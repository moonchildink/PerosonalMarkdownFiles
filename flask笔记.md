# flask 笔记







## 1.应用的基本结构

### 1.1 路由与视图函数

路由：

> 客户端（例如Web浏览器）把请求发送给Web服务器，Web服务器再把请求发送给Flask应用实例。应用实例需要知道对每个URL的请求要运行哪些代码，所以保存了一个URL到Python函数的映射关系。处理URL和函数之间关系的程序称为路由。

flask借助装饰器来处理路由。

```pyhton
def index():
    return '<h1>Hello World!</h1>'

app.add_url_rule('/', 'index', index)
```

其中，`index()`被称为**视图函数**，函数的返回值称为响应，是客户端接收到的内容。

+ **使用URL传递参数**，动态路由

```
# 传入参数的两种方式
@app.route("/blog/<int:blog_id")
def blog_detail(blog_id):
    return "您访问的博客是：%s"%blog_id


@app.route("/book/list")
def book_list():
    # argument:参数
    # request.args:参数字典
    page = request.args.get("page",default=1,type=int)
    return f"您获取的是第{page}页的图书列表"

```

+ **使用命令行工具启动flask服务器**

```
(venv) $ set FLASK_APP=hello.py
(venv) $ flask run
 * Serving Flask app "hello"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

### 1.2 请求-响应循环

1.   flask全局上下文

     ![image-20230811121402132](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230811121402132.png)

     `g`可用于临时存储相关变量，该变量全局可访问；

     `request`：用于封装客户端发送的HTTP请求报文

     `session`：会话？类似asp.net中的session吗:question:

     `current_app`：当前应用实例



### 1.3 Cookie

1.   流程
     1.   登录
     2.   服务端验证用户名和密码，设置好cookie后返回给浏览器
     3.   浏览器端保存cookie到本地
     4.   浏览器再次发送请求时会自动携带浏览器cookie
     5.   服务端取出对应cookie，返回相应的用户数据
2.   特点
     1.   客户端会话技术
     2.   数据存储在客户端之中
     3.   浏览器再访问时会自动携带当前cookie
     4.   不能跨域、跨浏览器
3.   实现
     1.   设置cookie：`response.set_cookir(key,value,)`



## 2. 基于jinja的模板

（目前来说暂时用不到，因为准备做前后端分离）





## 3. 数据库

### 3.1 数据库初始设置与测试

首先安装mysql8.0，之后使用Navicat链接mysql数据库，创建数据库。

在app.py之中输入如下代码：

```
app = Flask(__name__)

# 在app.config之中设置好数据库信息
# 传递app时，可以直接连接数据库
HOSTNAME  = '127.0.0.1'         # 设置数据库的IP地址
PORT='3306'
USERNAME = 'root'
PASSWORD = "112358"
DATABASE = "new_database"

app.config['SQLALCHEMY_DATABASE_URI'] = \
    f"mysql+pymysql://{USERNAME}:{PASSWORD}@{HOSTNAME}:{PORT}/{DATABASE}?charset=utf8mb4"


db = SQLAlchemy(app)
with app.app_context():
    with db.engine.connect() as conn:
        rs = conn.execute(text("select 1"))       # 执行sql语句
        print(rs.fetchone())                # 应该输出（1，）

```

**注意：**如果直接执行`rs = conn.execute("select 1")`报错，使用`conn.execute(conn("select 1"))`即可。

:question:：为什么此处需要使用`with app.app_context()`来请求应用的上下文？

**数据库初始化：**

首先在Mysql Workbench之中创建相应的数据库（Schema）；

`app/__init__.py`:首先创建`SQLAlchemy()`对象，之后在工厂函数中执行：`db.init_app(app)`

`flasky.py`：`with app.app_context(): db.creat_all()`

之后会自动检索在`model.py`之中的数据库模型，之后创建tables.

### 3.2 ORM模型

ORM:对象关系映射(object relationship mapping)，可以使用python面向对象的方式来连接数据库。而不需要写原生的sql语句。

#### 3.2.1 SQLAlchemy基础

1. 连接数据库：

![image-20230321215349677](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230321215349677.png)

2. 在flask之中链接mysql数据库

```python
HOSTNAME = '127.0.0.1'  # 设置数据库的IP地址
PORT = '3306'
USERNAME = 'root'
PASSWORD = "112358"
DATABASE = "new_database"

app.config['SQLALCHEMY_DATABASE_URI'] = \
    f"mysql+pymysql://{USERNAME}:{PASSWORD}@{HOSTNAME}:{PORT}/{DATABASE}?charset=utf8mb4"
```

3. 创建SQLAlchemy对象：`db = SQLAlchemy(app)`
4. 定义模型

```python
class User(db.Model):
    __tablename__ = "user"
    """
    autoincrement:自动增长，
    nullable：不可为空
    """
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)  # 创建一个列字段
    username = db.Column(db.String(100), nullable=False)
    password = db.Column(db.String(100), nullable=False)
    
```

```python
class User(db.Model):
    __tablename__ = "user"
    """
    autoincrement:自动增长，
    nullable：不可为空
    """
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)  # 创建一个列字段
    username = db.Column(db.String(100), nullable=False)
    password = db.Column(db.String(100), nullable=False)
```

5. 常用的列数据类型

![image-20230321215641459](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230321215641459.png)

6. 常用的列参数

![image-20230321220610698](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230321220610698.png)

#### 3.2.2 关系

```python
class Role(db.Model):
    # ...
    users = db.relationship('User', backref='role')

class User(db.Model):
    # ...
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))

```

> 关系使用users表中的外键连接两行。添加到User模型中的role_id列被定义为外键，就是这个外键建立起了关系。传给db.ForeignKey()的参数'roles.id'表明，这列的值是roles表中相应行的id值。
>
> 从“一”那一端可见，添加到Role模型中的users属性代表这个关系的面向对象视角。对于一个Role类的实例，其users属性将返回与角色相关联的用户组成的列表（即“多”那一端）。db.relationship()的第一个参数表明这个关系的另一端是哪个模型。如果关联的模型类在模块后面定义，可使用字符串形式指定。

*这一部分略难*

+ `relationship()`之中的参数说明。

![image-20230321222033231](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230321222033231.png)



#### 3.2.3 基于ORM进行基础的增删改查

```python
@app.route("/auth/user")
def add_user():
    # 1. 创建ORM对象
    user = User(username="moonchild",password="11111111")
    # 2. 添加到session之中
    db.session.add(user)
    # 3. 同步到数据库之中
    db.session.commit()
    return "用户创建成功！"


# 查找数据
@app.route("/auth/query")
def query_user():
    # 1.get查找:通过主键进行查找
    user = User.query(1)
    print(f"{user.id}:{user.username}--{user.password}")
    # return "数据查找成功！"
    # 2. 通过filter_by进行查找
    users = User.query.filter_by(username="moonchild")  # 返回Query对象，可以进行迭代。for ... in...
    print(type(users))
    return "数据查找成功"

# 更新数据
@app.route("/auth/update")
def update_user():
    user = User.query.filter_by(username="moonchild").first()
    user.password = "22222222"
    db.session.commit()
    return "数据修改成功！"

# 删除数据
@app.route("/auth/delete")
def delete_user():
    user = User.query.filter_by(username="moonhcild").first()
    db.session.delete(user)
    # 将在session之中的删除操作同步到数据库之中
    db.session.commit()
    return "数据查找成功！"
```





## 4. 工程化

### 4.1 项目结构以及配置

#### 4.1.1 项目结构

```
|-flasky
  |-app/
    |-templates/
    |-static/
    |-main/				# main blueprint
      |-__init__.py
      |-errors.py
      |-forms.py
      |-views.py
    |-__init__.py
    |-email.py
    |-models.py
  |-migrations/
  |-tests/
    |-__init__.py
    |-test*.py
  |-venv/
  |-requirements.txt
  |-config.py
  |-flasky.py
```

#### 4.1.2 项目配置





### 4.2 蓝本

#### 4.2.1 为什么需要蓝本

1.   对于应用包（app），需要使用工厂函数，对其进行定制的初始化。
2.   封装模块内容

可以在注册蓝本的`register_blueprint`函数中，使用`subdomain`参数来指定子域名，比如`api.example.com`









### 钩子函数

也叫中间件

+   `app.before_first_request`：部署第一次请求之前执行
+   `app.before_request`：每次响应请求之前执行：可以在此函數之中，添加session或者flask_login判斷
+   `blueprint.context_processor`：
+   `blueprint.errorhandler`：處理錯誤，可以寫成`errorhandler(404),errorhandler(500)`

![image-20230817101455963](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230817101455963.png)

>   实际部署中，WebAPI可以作为单独的程序，也可以和传统的Flask程序进行组合，比如使用传统模式编写认证系统，编写API返回资源



+   添加跨域支持
+   识别客户端类型
+   





## SQL语句

创建数据库：`CREATE SCHEMA `new_database` DEFAULT CHARACTER SET utf8mb4 ;`

数据库查询：`SELECT column1,column2... FROM  schema.table WHERE condition `





## 开发设计

**数据库设计**

Role

-   Administrator
-   User
-   Default



**添加跨域支持**

```python
from flask_cros import CROS

CROS(api_blueprint)			# 将该部分代码添加到蓝本的__init__文件之中即可
```





**功能模块**

-   用户注册、登录以及验证，主要是实现识别当前用户。似乎在uniapp前端也包含本地cookie
-   手语识别，这个类似直播推流
-   



