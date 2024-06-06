# Spring 笔记

相关博客：

[paperfly's blog](https://www.paperflying.com/2023/05/21/%E9%A1%B9%E7%9B%AE%E7%AC%94%E8%AE%B0/%E9%BB%91%E9%A9%AC%E7%82%B9%E8%AF%84/%E9%BB%91%E9%A9%AC%E7%82%B9%E8%AF%84%E9%9D%A2%E8%AF%95%E5%8F%AF%E8%83%BD%E7%9A%84%E9%97%AE%E9%A2%98%E6%95%B4%E7%90%86/#%E7%BC%93%E5%AD%98%E6%9B%B4%E6%96%B0%E9%97%AE%E9%A2%98)

[kyle's blog](https://cyborg2077.github.io/2022/10/22/RedisPractice/#%E7%BC%93%E5%AD%98%E9%9B%AA%E5%B4%A9%E9%97%AE%E9%A2%98%E5%8F%8A%E8%A7%A3%E5%86%B3%E6%80%9D%E8%B7%AF)

## 项目笔记



Redis缓存内存更新策略：

1. 内存淘汰：Redis内存淘汰机制：适用于低一致性需求
2. 超时剔除：给缓存数据添加TTL时间，到期删除缓存。下次查询时更新
3. 主动更新：修改数据库时更新缓存。

主动更新的实现方案：

1. Cache Aside Pattern：由调用者同时更新
2. Read/Write Through pattern：缓存与数据库整合为一个服务，由服务来维护一致性。调用者无需关心一致性，对外透明
3. Write Behind Caching Pattern：调用者之操作缓存，其他线程异步将数据持久化到数据库，保持最终一致

缓存数据库一致性：

1. 先删缓存，更新数据库：删除缓存，更新数据库；查询数据库，存入缓存。
2. 先更新数据库，在删缓存：更新数据库，删除缓存；查询缓存未命中，未命中，查询数据库，更新缓存。（**发生错误的概率更低**）
3. 异常情况：脏读，更新数据库以后另一线程来查询缓存，得到错误数据；解决方案：保持数据库与缓存操作的原子性，分布式事务锁。



#### 缓存穿透

1. 什么是缓存穿透：客户端请求的数据在客户端和缓存中都不存在的情况，缓存永远不会生效，这些请求会发送到数据库。可能会对数据库产生较大负担，而且当远程数据库会拖慢访问速度。

2. 解决方案：

   1. 缓存空对象：缓存null到Redis，设置TTL(Time to live)。可能会存在短期不一致性
   
   2. 布隆过滤：在客户端和Redis之间添加一层布隆过滤器，如果不存在（数据库与Redis）拒绝，否则放行。
      1. 实现复杂
      2. 存在误判的可能
   
   3. 增加ID的复杂度，进行数据格式校验，加强用户权限管理，热点参数的限流

   4. **互斥锁**：类似操作系统中的并行处理。考虑到缓存重建的资源消耗可能较大，且存在多个线程同时存入相同缓存的情况。但是耗费过大：一个线程获取锁以后其他线程可能等待较长时间，且存在可能产生死锁
   
      1. 实现逻辑：使用Redis的`SETNX`命令：set the value of a key,only if the key doesn't exist.对用Java中的`setIfAbsent`函数
   
      2. 代码
   
         ```java
         String shopJson = template.opsForValue().get(SystemConstants.REDIS_CACHE + id);
                 Shop shop = null;
                 if (shopJson == null) {
                     // 说明不存在于Cache之中,进行缓存重建
                     String lockKey = "lock:shop:" + id;
                     try {
                         boolean isSuccess = tryLock(lockKey);
                         if (!isSuccess) {
                             Thread.sleep(50);
                             // 重试查询
                             return queryWithMutex(id);
                         }
                         // 获取成功,查询数据库，存入Redis，释放lock
                         shop = getById(id);
         
                         // 模拟数据库重建延迟
                         Thread.sleep(200);
                         if (shop == null) {
                             // 说明也不存在于数据库之中,那么在Cache中设置一个空字符串
                             template.opsForValue().set(SystemConstants.REDIS_CACHE + String.valueOf(id), "", 3L, TimeUnit.MINUTES);
                             return null;
                         }
                         template.opsForValue().set(SystemConstants.REDIS_CACHE + id, JSONUtil.toJsonStr(shop), 30L, TimeUnit.MINUTES);
                     } catch (InterruptedException e) {
                         throw new RuntimeException(e);
                     } finally {
                         releaseLock(lockKey);
                     }
                     return shop;
                 }
                 // 在缓存之中
                 // 1. 空对象
                 if (shopJson.equals("")) {
                     return null;
                 }
                 return JSONUtil.toBean(shopJson, Shop.class);
         
         
         private boolean tryLock(String key) {
                 Boolean block = template.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
                 return BooleanUtil.isTrue(block);
             }
         
             private boolean releaseLock(String key) {
                 return Boolean.TRUE.equals(template.delete(key));
             }
         ```
   
         
   
         ![image-20240411210921292](https://raw.githubusercontent.com/moonchildink/image/main/imgs202404112109342.png)
   
   5. **逻辑过期**：
   
      1. 缓存是永久存在于数据库的，设置的TTL是**逻辑过期时间**。（实际上，再存入到Redis中时未设置过期时间，但是添加了一位数值，即逻辑TTL，查询结果超过logical TTL执行重新从数据库中获取数据存入缓存）。
   
      2. 实现方案：
   
         ![image-20240411215819861](https://raw.githubusercontent.com/moonchildink/image/main/imgs202404112158916.png)
   
   
   ![image-20240411210525366](https://raw.githubusercontent.com/moonchildink/image/main/imgs202404112105509.png)

#### 缓存雪崩

1. 什么是缓存雪崩：同一时段大量Key失效，或者Redis服务宕机，导致大量请求到达数据库，带来巨大压力。
2. 解决方案：
   1. 对TTL加入随机值
   2. 利用Redis集群提高服务的可用性
   3. 给缓存业务添加降级限流策略
   4. 添加多级缓存



#### 缓存击穿

1. 热点key失效，大量请求忽然到达数据库，造成服务器压力
2. 解决方案：
   1. 悲观锁互斥
   2. 设置逻辑过期方案，不设置实际的过期时间，比如`template.opsForValue().set(key, "", SystemConstants.REDIS_NULL_TTL, TimeUnit.MINUTES);`，也就是数据永久缓存在Redis之中；添加一个ttl字段，如果判断数据超过了ttl，那么先返回旧数据，新建线程异步更新缓存



![image-20240519152015263](https://raw.githubusercontent.com/moonchildink/image/main/imgs202405191520341.png)



### 秒杀系统

![image-20240519144918142](https://raw.githubusercontent.com/moonchildink/image/main/imgs202405191449250.png)

#### 生成全局唯一ID

1. 使用64位记录，组成：1bit符号位，31bit时间戳，32bit索引位。
2. 方法：首先生成31bit时间戳，之后使用redis increase自增操作生成索引
3. 拼接后返回ID
4. 生成ID的其他方法：雪花ID，UUID，数据库自增

#### 实现优惠券秒杀下单

- ==解决超卖问题==：多线程并发场景下，会产生超卖问题。

1. 悲观锁：认为线程安全问题一定会发生，在操作之前获取锁，确保线程串行执行.如`Synchronized`或者`Lock`
   1. 优缺点：简单粗暴但是性能一般
2. 乐观锁：认为线程安全问题不一定会发生，因此不加锁，只有在**更新数据时**判断有没有修改数据
   1. 判断方法：
      - 版本号法：给column添加version字段，更新时判断version是否正确
      - CAS(Compare and Swap)方法：进行更新时查询指定字段是否发生变化，发生变化拒绝更新
   2. 缺点：成功率较低
3. `AopContext.currentProxy()`代理的使用：在使用`@Transactional`注解声明事务时，可能存在以下情况：
   1. 不同类中，事务方法A调用非事务方法B，事物具有传播性，事务正常执行
   2. 不同类中，非事务方法A调用事务方法B，事务生效；
   3. 同一个类中，事务方法A调用非事务方法B，事务生效；
   4. **同一个类中，非事务方法A调用事务方法B，事务失效，由于使用Spring AOP代理导致的，只有当事务方法被当前类以外的代码调用时，才会有Spring生成的代理对象来管理**


#### 一人一单

1. `synchronized(userId.toString().intern())`：根据常量池中的字符串值来加锁，保证同一userid的用户使用同一把锁，否则`toString()`方法将返回不同的`String`对象

2. 使用`@Transcational()`是数据库事务需要整体加锁，否则可能事务失效

3. 对事务加锁：

   ```java
   Long userId = UserHolder.getUser().getId();
           synchronized (userId.toString().intern()) {
               // 对代理对象加锁
               IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
               return proxy.createVoucherOrder(voucherId);
           }
   ```

4. **分布式锁**

   1. 为什么要使用分布式锁：使用传统的JVM提供的`synchronized`关键字，锁仅仅在单进程可见，对于服务集群情况下依旧会产生并发安全问题。因此，我们需要使用分布式锁，目的是为了实现多台服务器可见。**多进程使用同一把锁**。

   2. 特点：**多进程可见** **互斥** 高可用 高性能 安全性

   3. 实现方案：

      1. MySQL
      2. Redis
      3. Zookeeper：利用节点的唯一性和有序性实现互斥

   4. 基于Redis的实现方案：`set lock thread1 nx ex ttl`，`nx`表示互斥，`ex`表示过期时间。但是仅仅使用`setex`命令还存在*误删锁*的问题

      ![image-20240413232812375](https://raw.githubusercontent.com/moonchildink/image/main/imgs202404132328465.png)

      然而，在上图中的情况表明，**可能存在线程执行结束释放其他线程锁的情况**，显然会导致数据错误。

      解决方案：使用UUID+ThreadID生成唯一的Value，根据Value判断是否可以释放锁。

      ![image-20240413234507915](https://raw.githubusercontent.com/moonchildink/image/main/imgs202404132345049.png)

      上图的情况又表明，如果判断锁标识语句与删除锁语句之间存在间隔，也就是可能产生“*超时释放*”可能会导致删除的错误。因此，需要保证这两个动作的**原子性**

      1. 然而基于setnx实现的分布式锁依旧存在以下问题：
         1. 不可重入：同一个线程无法多次获取同一把锁(为什么要获取同一把锁？)
         2. 不可重试：获取锁只会尝试一次，没有重试机制
         3. 超时释放
         4. 主从一致

   5. 改进方法：使用Redisson

      1. 执行逻辑

         ![image-20240423212416399](https://raw.githubusercontent.com/moonchildink/image/main/imgs202404232124482.png)

      2. Question

         1. 为什么要设置锁重入？- 同一线程可能多次执行需要互斥性的业务，比如线程递归调用
         2. 为什么要设置计数？-避免误删锁的情况
         3. 锁设置多少TTL合理？-Redisson默认的锁超时时间是30seconds

      3. Redisson的特性：

         1. 可重入：使用hash结构记录线程id和重入次数
         2. 可重试：使用信号量和PubSub机制实现等待、唤醒，获取锁失败的重试机制
         3. 超时释放：利用watchDog，每隔一段时间充值超时时间
         4. 主从一致性：
            1. 产生原因：主节点失效时未完成向从节点消息同步
            2. 解决方法：MultiLock

5. 总结：

   1. 不可重入式Redis分布式锁
      1. 原理：利用`setnx`的互斥性；利用`ex`避免死锁；释放锁时判断线程表示
      2. 缺点：不可重入、无法重试、超时失效
   2. 可重入式Redis分布式锁
      1. 利用Hash结构，记录线程标识和重入次数；利用watchDog延续锁时间；利用信号量控制锁重试和等待
      2. 缺点：Redis宕机导致锁失效问题
   3. Redisson的multiLock
      1. 多个独立的Redis节点，必须在所有节点都获取重入锁，才算获取成功
      2. 缺点：实现复杂，成本高、

6. 抢购流程优化

   1. 将判断库存和写入订单的操作利用Redis进行，这一部分使用Lua脚本来操作，目的是为了**操作的原子性**

      ```lua
      local voucherId = ARGV[1]
      local userId = ARGV[2]
      
      local stockKey = 'seckill:stock:' .. voucherId
      local orderKey = 'seckill:order:' .. voucherId
      
      
      
      -- 判断库存
      if (tonumber(redis.call('get', stockKey)) <= 0) then
          return 1
      end
      
      -- 判断用户是否下单
      if (redis.call('sismember', orderKey, userId) == 1) then
          -- 说明重复下单
          return 2
      end
      
      -- 表示库存充足而且用户没有购买过
      -- 执行减库存
      redis.call('incrby', stockKey, -1)
      -- 指定记录用户ID
      redis.call('sadd', orderKey, userId)
      
      return 0
      
      ```

   2. 修改优惠券秒杀的逻辑，新的逻辑描述如下：

      1. 首先执行Lua脚本，判断用户是否已经购买（使用Set结构存储指定优惠券对应的用户ID），判断当前优惠券库存是否大于零
      2. 如果满足未购买、有库存条件，则：
         1. 将创建订单的操作使用消息队列来处理
      3. 当前逻辑的问题：存在内存安全问题（使用了就JVM提供的阻塞队列），数据安全问题

   3. 使用Redis消息队列完成异步订单处理（**消息队列的用处：异步、解耦、削峰**）
   
      1. Redis消息队列结构：
   
         1. List：基于List结构模拟消息队列
   
         2. PubSub：点对点消息模型
   
         3. Stream：
   
      2. List结构（双向链表），实现方法：结合`BLPUSH`与`BRPOP`命令，`B`表示阻塞，阻塞等待。可以实现阻塞队列的效果。但是可能存在**消息丢失**的情况，而且只支持**单消费者**的情况
   
      3. PubSub机制：
   
         1. `SUBSCRIBE channel`:定于一个或者多个频道
         2. `PUBLISH channel msg`：向一个频道发送消息
         3. `PSUBSCRIBE pattern[]`：定于符合pattern格式的所有频道的消息
         4. 优点：支持发布订阅（多生产多消费模式）、支持阻塞
         5. 缺点：**不支持消息持久化，消息堆积有上限，超出时消息丢失**
   
      4. 基于Stream的消息队列
   
         1. `XADD key [[NOMKSTREAM] MAXLEN|MINID [=|~] threshold [LIMIT count]] *|ID field value`
   
            ![image-20240518233057681](https://raw.githubusercontent.com/moonchildink/image/main/imgs202405182330768.png)
   
         2. `XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...] `，**但是XREAD可能存在问题：指定起始ID为$时，戴白哦读取最新的消息，如果处理一条消息的过程中，又有超过1条以上的消息到达队列，则下次只能获取最新的一条，出现*漏读消息*的问题**
   
            ![image-20240518233433395](https://raw.githubusercontent.com/moonchildink/image/main/imgs202405182334471.png)
   
         3. 优点：消息可回溯；可以被多个消费者读取；可以阻塞读取
   
         4. 缺点：有消息漏读的风险
   
      5. **消费者组**：
   
         1. 消息分流：分流给不同的消费者，来加快处理速度
   
         2. 消息标识：确保消息得到消费，消费者组会维持一个标志记录最后被处理的消息，哪怕服务宕机也会从表示之后处理消息，确保得到处理。
   
         3. 消息确认机制：消费者获取到消息以后，消息处于pending状态，并且存入到pending-list之中。当处理结束以后，需要通过XACK机制来确认消息，标记为已处理，才会从pending-list之中移除
   
         4. 创建消费者组：`XGROUP CREATE key groupName ID [MKSTREAM]`
   
            ![image-20240518234427841](https://raw.githubusercontent.com/moonchildink/image/main/imgs202405182344925.png)
   
            ![image-20240518234654137](https://raw.githubusercontent.com/moonchildink/image/main/imgs202405182346217.png)
   
         5. 优点
         
            1. 消息可回溯
            2. 可以多消费者争抢消息，加快速度
            3. 可以阻塞读取
            4. 没有消息漏读的风险
            5. 有消息确认机制，保证消息被消费
      
      ![image-20240519105541464](https://raw.githubusercontent.com/moonchildink/image/main/imgs202405191055587.png)

### 秒杀优化

![image-20240518224534379](https://raw.githubusercontent.com/moonchildink/image/main/imgs202405182245519.png)

1. 在Redis中保存库存信息和已购买用户信息，这一过程使用Lua脚本来保证其**原子性**
2. 将扣减库存操作放到阻塞队列之中，使用新线程异步处理写入数据库操作



### 关注推送的实现

1. 关注推送也叫Feed流，直译为投喂。

2. 常见模式：

   1. timeline：不做内容筛选，简单按照内容发布时间排序
   2. 智能排序：使用派速算法屏蔽违规、用户不感兴趣的内容。推送用户感兴趣的信息

3. 实现方案

   1. 拉模式：延迟高

   2. 推模式（写扩散）：缺点为内存占用较高，同一份消息需要推送给较多的用户

   3. 推拉结合：

      ![image-20240425130422189](https://raw.githubusercontent.com/moonchildink/image/main/imgs202404251304318.png)

	![image-20240425130440628](https://raw.githubusercontent.com/moonchildink/image/main/imgs202404251304694.png)

4. 实现
   1. 新增笔记时推送到粉丝收件箱
   2. 收件箱按时间戳排序
   3. 使用分页查询收件箱数据







## 项目拷打

好的，我们可以进一步深入技术细节，并且考虑到一些潜在的缺陷或问题。以下是更深入的面试问题：

### SpringBoot 相关问题
1. **SpringBoot 的核心功能**：
   - 你提到使用 SpringBoot 开发项目，能具体讲一下你是如何组织 SpringBoot 项目结构的吗？
   - 你在 SpringBoot 项目中是如何管理依赖的？是否遇到过依赖冲突问题，如何解决的？

2. **SpringBoot 配置管理**：
   - 请描述一下 SpringBoot 的自动配置原理。你在项目中是否自定义过自动配置，如何实现的？

### Redis 相关问题
3. **缓存穿透**：
   - 请详细解释空置法和布隆过滤器在解决缓存穿透中的具体实现细节。你是如何选择布隆过滤器的参数（如哈希函数数量和位数组大小）的？

4. **缓存击穿与缓存雪崩**：
   - 你提到使用互斥锁解决缓存击穿问题，能详细描述一下互斥锁的具体实现方式吗？会不会对性能产生影响？
   - 请详细讲解一下你是如何进行热点缓存异步更新的，如何保证更新的及时性和一致性？

5. **Redis + JWT 实现无状态分离**：
   - 在 Redis 中存储 JWT 的优缺点有哪些？如何解决 Redis 单点故障问题？

6. **Redis 和 RabbitMQ 的结合使用**：
   - 请详细讲述一下你是如何设计 Redis 和 RabbitMQ 的结合使用来处理高并发下的用户下单流程的。如何处理消息的幂等性和重复消费问题？

7. **Redisson 分布式锁**：
   - 你提到使用 Redisson 分布式锁来优化秒杀，能详细讲解一下 Redisson 分布式锁的实现原理吗？
   - 在使用 Lua 脚本时，有没有考虑过脚本执行超时的问题？如何处理？

### MyBatis Plus 相关问题
8. **MyBatis Plus 优势**：
   - 请详细讲解 MyBatis Plus 的一些高级功能，例如自动填充、逻辑删除和多租户支持，你在项目中是如何使用这些功能的？
   - 在使用 MyBatis Plus 时，是否遇到过性能问题，如何优化的？

### RabbitMQ 相关问题
9. **消息队列的使用场景**：
   - 请详细描述一下 RabbitMQ 的交换机类型及其使用场景，你在项目中选择了哪种类型的交换机？为什么？
   - 在高并发情况下，RabbitMQ 消息积压如何处理？你是如何监控和优化 RabbitMQ 的性能的？

### 分布式系统相关问题
10. **分布式锁**：
    - 除了 Redisson，你还了解哪些其他实现分布式锁的方式？请详细比较它们的优缺点。
    - 在分布式系统中，如何避免锁的死锁和活锁问题？

11. **流量削峰填谷**：
    - 详细讲述一下你是如何通过 Redis 和 RabbitMQ 实现流量削峰填谷的。如何保证消息不丢失？
    - 在流量高峰期，如何动态调整系统资源以应对突发流量？

### 其他问题
12. **跨域请求**：
    - **请详细解释一下跨域请求的原理。你在项目中是如何处理复杂跨域请求的？**
    - 自定义拦截器进行跨域请求拦截验证时，如何处理性能开销和安全问题？

13. **项目挑战与解决**：
    - 在这个项目中，你遇到的最大的技术难题是什么？你是如何解决的？
    - 项目上线后有没有遇到过性能瓶颈或者 bug？你是如何进行性能调优和 bug 修复的？

### 潜在缺陷与改进
14. **Redis 缓存问题**：
    - 在大规模应用中，Redis 缓存策略可能遇到哪些问题？你是如何解决缓存数据一致性和缓存过期策略的？
    - Redis 持久化方案（RDB 和 AOF）在你的项目中是如何选择和配置的？

15. **RabbitMQ 的可靠性**：
    - 在使用 RabbitMQ 时，如何保证消息传输的可靠性？如何处理消息丢失、重复和顺序性问题？
    - RabbitMQ 集群配置中有哪些注意事项？如何实现高可用？

通过这些问题，可以深入挖掘候选人对技术细节的理解和掌握程度，同时评估他们在实际项目中应对复杂问题和解决潜在缺陷的能力。



这段经历涵盖了多种技术和问题解决方案，是一个非常好的面试题材。以下是根据这段经历提问的一些问题：

### SpringBoot 相关问题
1. **SpringBoot 的核心功能**：
   - 请简要描述一下 SpringBoot 的核心功能和它的优点。
   - 你在项目中使用了哪些 SpringBoot 的特性或模块？

2. **SpringBoot 配置管理**：
   - 你是如何管理 SpringBoot 项目的配置文件的？请介绍一下 `application.properties` 或 `application.yml` 的使用。

### Redis 相关问题
3. **缓存穿透**：
   - 请解释一下什么是缓存穿透，以及你是如何使用空置法和布隆过滤器解决这个问题的？

4. **缓存击穿与缓存雪崩**：
   - 你提到使用互斥锁（悲观锁）和热点缓存异步更新来解决缓存击穿和缓存雪崩问题，能详细讲解一下具体实现方式吗？

5. **Redis + JWT 实现无状态分离**：
   - 请描述一下你是如何使用 Redis 和 JWT 来实现前后端无状态分离的。

6. **Redis 和 RabbitMQ 的结合使用**：
   - 在多线程并发环境下，你是如何利用 Redis 和 RabbitMQ 优化用户下单流程的？请详细讲述一下你的实现思路和步骤。

7. **Redisson 分布式锁**：
   - 你提到使用 Redisson 分布式锁实现秒杀优化，能详细讲解一下实现过程吗？
   - 为什么使用 Lua 脚本可以保证操作的原子性？

### MyBatis Plus 相关问题
8. **MyBatis Plus 优势**：
   - 相对于传统的 MyBatis，MyBatis Plus 提供了哪些增强功能？
   - 你在项目中具体是如何使用 MyBatis Plus 的？

### RabbitMQ 相关问题
9. **消息队列的使用场景**：
   - 你是如何在项目中利用 RabbitMQ 的？具体解决了什么问题？
   - 请解释一下你是如何处理 RabbitMQ 消息的生产和消费过程的。

### 分布式系统相关问题
10. **分布式锁**：
    - 除了 Redisson，你还了解其他实现分布式锁的方式吗？可以简单介绍一下。

11. **流量削峰填谷**：
    - 请详细讲解一下你是如何通过 Redis 和 RabbitMQ 实现流量削峰填谷的。

### 其他问题
12. **跨域请求**：
    - 你是如何实现跨域请求拦截验证的？具体使用了哪些技术？

13. **项目挑战与解决**：
    - 在这个项目中，你遇到了哪些主要的挑战？你是如何解决的？

通过这些问题，你可以深入了解候选人对各个技术栈的掌握程度，以及他们在实际项目中是如何应用这些技术的。这些问题不仅涵盖了基础知识，还涉及到具体的实现细节和解决方案，有助于评估候选人的实际开发能力和问题解决能力。
