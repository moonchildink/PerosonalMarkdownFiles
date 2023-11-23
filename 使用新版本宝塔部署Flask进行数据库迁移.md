# 使用新版本宝塔部署flask、进行数据库迁移

[TOC]

## flask部署

1.   首先我们需要将代码推送到Github或者Gitee上，这里推荐使用Gitee（后期使用`pip pull origin main`命令更新代码时由于网络问题无法从GitHub上拉取代码）。

2.   通过Xshell或者其他ssh命令行工具进入到Linux环境中，之后使用`pip clone`命令克隆代码到本地

3.   在宝塔面板中（如何在服务器上安装宝塔界面可以自行搜索），进入到当前界面：

     ![image-20230905134241027](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230905134241027.png)

4.   新建项目![image-20230905134509542](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230905134509542.png)

​	填入相关信息，端口可以填8000，框架选择flask，运行方式选择uwsgi。

1.   **依赖问题**：我在运行过程中按照requirements.txt安装依赖无效，在“设置-项目日志”中查看日志会报错"module ... not found"，此时解决方法有二：

     1.   点击项目右侧的模块：

          ![image-20230905134936021](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230905134936021.png)

​			![image-20230905134947163](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230905134947163.png)

依次安装所需模块即可

2.   在shell中安装：（新版本宝塔已经在虚拟环境中的bin目录里移除了active文件，所以无法通过`source active`命令激活虚拟环境）进入项目目录：

     ![image-20230905135212053](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230905135212053.png)

​		使用命令`319c3206a7f10c17c3b9116d4a957646_venv/bin/pip install scikit-learn`使用`pip`安装所		需依赖模块，依次安装即可。

之后在宝塔面板以及服务器中放行端口，即可访问。





## 数据库迁移

1.   按着上述方法进入到项目目录
2.   使用虚拟环境中的pip安装`flask`,`flask-migrate`以及`pymysql`模块
3.   使用命令`319c3206a7f10c17c3b9116d4a957646_venv/bin/flask db init`进行迁移数据初始化，如果此处有migrates文件夹，请使用`rm -rf`指令删除。
4.   同样使用`db migrate`,`db upgrade`命令进行数据库迁移

![image-20230905140319905](https://cdn.jsdelivr.net/gh/moonchildink/image@main/imgs/image-20230905140319905.png)