# Java基础

## 数据类型

==包装器类型==的`equals()`和`==`

1. 基本数据类型，9种(都是有符号)，对于大数可以使用`BigInteger`以及`BigDecimal`

   ![image-20240418171547708](https://raw.githubusercontent.com/moonchildink/image/main/imgs202404181715774.png)

2. 内存分配区域

   1. 寄存器：对程序员透明
   2. 堆栈：在堆栈中的元素具备确切的生命周期
   3. 堆：通用内存池，存放所有的Java对象。编译器无需知道数据在堆中的存活时间，具备很大的灵活性。
   4. 常量

3. 方法、属性

   1. 方法参数中传递的是引用

4. static关键字：修饰**字段与方法**，没有`this`的方法，在`static`方法内部不能调用非静态方法

5. 字符串操作符`+`和`+=`

   1. `StringBuilder`内部使用`char[]`来存储字符，且在初始化时定义长度
   2. 使用拼接运算符时，Java实际上对字符串修改为`StringBuilder`类，并调用`append`方法进行字符串拼接。然后调用`toString`方法
   3. 使用`+`会频繁出现扩容以及赋值，造成资源的消耗

6. 类型转换：窄化操作会造成数据丢失（总是执行**截断**操作，向下转换，不会四舍五入），拓展转换不会造成数据丢失，不必显式进行转换





## 初始化和清理

1. Java垃圾回收器负责回收无用**对象**占据的内存资源，也就是堆区内存。
2. `finalize()`函数：允许在类中定义`finalize()`方法，一旦垃圾回收器准备释放对象占用的空间时，首先调用对象的`finalize()`方法，并且在下一次垃圾回收动作发生时，才会真正回收对象占用的内存。但是，**对象可能不被垃圾回收；垃圾回收不等于析构函数**。
3. 垃圾回收器在工作时，会*一边回收空间，一边紧凑对象排列*，这样可以避免频繁的页面调度。
4. 垃圾回收器的实现：不同的JVM实现了不同的垃圾回收技术，有一种名为”stop and copy“的方法：暂停程序的运行，将所有存活对象从一个堆复制到另一个堆，未被复制的便是垃圾。新堆对象紧密排列。
5. 初始化顺序：
   1. 首次实例化对象时，或者静态方法/静态域首次被访问时，解释器会查找类路径
   2. 载入相应`.class`文件，执行有关静态初始化的动作
   3. 创建对象时，分配堆空间，将该块空间清0
   4. 变量会在任意方法调用之前进行初始化；
   5. 静态数据只占用一份内存区域，会在Class对象首次加载时执行一次

6. 显式的静态初始化
   1. 将多个静态初始化动作组织为一个“静态子句”
   2. 静态子句在首次生成类对象时，或者首次访问静态数据时执行，且只执行一次

7. 非静态实例的初始化：调用任意显式构造器都会执行
8. 可变参数列表：`void function(Object... objs)`,`void function(String... strs)`，可以传递多个符合要求的参数，也可以传递一个数组。也可以什么参数都不传递。可变参数列表可以传递基本类型，也可以传递对应的包装类型。当基本类型和包装类型混合到一起时，自动包装机制会有选择地将`int`提升为`Integer`。



## 访问权限控制

1. `package`和`import`，package给出了一种分割命名空间的方法

## final和static

1. `final`具有如下作用：

   1. 修饰一个基本变量时，表示该变量是不可变变量，不允许修改其数值
   2. 修饰一个引用时，不可改变该引用指向的地址
   3. 修饰函数时，表示该函数不可被继承（对于`private`函数，由于该函数不会对外暴露，无法体现出`final`的影响）
   4. 修饰类时，表示该类为不可继承

   







## 容器

![image-20240426210033503](https://raw.githubusercontent.com/moonchildink/image/main/imgs202404262100603.png)\

---|Collection: 单列集合
			---|List: 有存储顺序, 可重复
				---|ArrayList:	数组实现, 查找快, 增删慢
					        由于是数组实现, 在增和删的时候会牵扯到数组
                                                增容, 以及拷贝元素. 所以慢。数组是可以直接
                                                按索引查找, 所以查找时较快
				---|LinkedList:	链表实现, 增删快, 查找慢
					        由于链表实现, 增加时只要让前一个元素记住自
                                               己就可以, 删除时让前一个元素记住后一个元
                                               素, 后一个元素记住前一个元素. 这样的增删效
                                             率较高但查询时需要一个一个的遍历, 所以效率
                                                较低
				---|Vector:	和ArrayList原理相同, 但线程安全, 效率略低
					         和ArrayList实现方式相同, 但考虑了线程安全问
                                                题, 所以效率略低
			---|Set: 无存储顺序, 不可重复
				---|HashSet
				---|TreeSet
				---|LinkedHashSet
---| Map: 键值对
		---|HashMap
		---|TreeMap
		---|HashTable
		---|LinkedHashMap

### 线性结构

1. `Stack`: 继承自`Vector`，底层使用数组实现。其效率较低

2. `ArrayDeque`：底层基于动态数组实现，效率相比较于Stack更高

   `arraydeque`的扩容操作：

   ```java
   private void grow(int needed) {
           // overflow-conscious code
           final int oldCapacity = elements.length;
           int newCapacity;
           // Double capacity if small; else grow by 50%
           int jump = (oldCapacity < 64) ? (oldCapacity + 2) : (oldCapacity >> 1);
           if (jump < needed
               || (newCapacity = (oldCapacity + jump)) - MAX_ARRAY_SIZE > 0)
               newCapacity = newCapacity(needed, jump);
           final Object[] es = elements = Arrays.copyOf(elements, newCapacity);
           // Exceptionally, here tail == head needs to be disambiguated
           if (tail < head || (tail == head && es[head] != null)) {
               // wrap around; slide first leg forward to end of array
               int newSpace = newCapacity - oldCapacity;
               System.arraycopy(es, head,
                                es, head + newSpace,
                                oldCapacity - head);
               for (int i = head, to = (head += newSpace); i < to; i++)
                   es[i] = null;
           }
       }
   ```

   





### JVM

#### 运行时数据区

1. 运行时数据区的结构：

   <img	src="https://pdai.tech/images/jvm/jvm/0082zybply1gc6fz21n8kj30u00wpn5v.jpg" >

2. 线程私有：程序计数器、虚拟机栈、本地方法区

3. 线程共享：堆、方法区、堆外内存（JDK7的永久代或JDK8的元空间、代码缓存）

4. PC：程序指针计数器，存储在寄存器中？

5. **虚拟机栈**：保存方法的局部变量、中间结果，参与方法的返回和调用。类似x86汇编中的函数调用、跳转，**显然不存在垃圾回收问题。**可能会出现`StackOverflow`异常（采取固定大小的虚拟机栈不满足需求时）和`OutOfMemoryError`异常（虚拟机栈动态扩展）

6. 单位为**栈帧（Stack Frame)**

7. 栈帧的内部结构：

   <img src="https://pdai.tech/images/jvm/jvm/0082zybply1gc8tjehg8bj318m0lbtbu.jpg" >

   1. 局部变量表：存储**方法参数和定义在方法体内部的局部变量、对象引用**，其大小在编译器确定，方法运行期间不会改变局部变量表的大小。

   2. 槽Slot：Slot是局部变量表的基本单位，32位以内的的类型占用一个Slot，64位的类型占用2个Slot

      1. `byte`,`short`,`char`在存储前被转换为`int`，`boolean`被转换为`int`
      2. `long`和`double`占用两个Slot
      3. JVM为局部变量表的每一个Slot分配一个访问索引，通过索引可以访问到指定变量
      4. **局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或者间接引用的对象都不会被回收**

   3. 操作数栈：每个栈帧中除了包含局部变量表以外还包含一个栈结构，称为表达式栈。

      1. 保存中间结果，同时作为变量临时存储空间
      2. JVM执行引擎的工作区，执行方法时会创建操作数栈，此方法的操作数栈是空的
      3. 单位深度为32bit
      4. 使用标准的pop/push操作来访问数据
      5. **jvm解释引擎是基于栈的**

   4. 动态链接：指向运行时常量池的方法引用

      <img src="https://pdai.tech/images/jvm/jvm/0082zybply1gca4k4gndgj31d20o2td0.jpg" >

      1. 



