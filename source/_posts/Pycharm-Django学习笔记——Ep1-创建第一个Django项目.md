---
title: Pycharm+Django学习笔记——Ep1.创建第一个Django项目
date: 2019-12-08 21:24:35
index_img: /img/Xnip2019-12-08_21-35-43.png
banner_img: /img/Xnip2019-12-08_21-35-43.png
tags: [Python,Django]
categories: 代码
---

### Pycharm新建项目

> 文件-新建项目

<!-- more -->

<img src="/img/Xnip2019-12-08_20-38-57.png" style="zoom:50%;" />

> 选择Django-填写项目名称-填写App名称

![](/img/Xnip2019-12-13_10-00-59.png)

> 点击CREATE

在首次创建项目的时候可能会报错`pip install django出错`

此时只需手动在Terminal中输入

```sh
pip3 install django
```

安装django就可以了

> 创建好的项目目录及含义如下

```python
blog                #APP应用名和目录
│  admin.py        #对应应用后台管理配置文件。
│  apps.py         #对应应用的配置文件。
│  models.py       #数据模块，数据库设计就在此文件中设计。后面重点讲解
│  tests.py        #自动化测试模块，可在里面编写测试脚本自动化测试
│  views.py        #视图文件，用来执行响应代码的。你在浏览器所见所得都是它处理的。
│  __init__.py
│
├─migrations        #数据迁移、移植文目录，记录数据库操作记录，内容自动生成。
│  │  __init__.py
myblog               #项目配置目录
│  __init__.py       #初始化文件，一般情况下不用做任何修改。
│  settings.py        #项目配置文件，具体如何配置后面有介绍。
│  url.py             #项目URL设置文件，可理解为路由，可以控制你访问去处。
│  wsgi.py          #为Python服务器网关接口，是Python与WEB服务器之间的接口。
myblogvenv            #Pycharm创建的虚拟环境目录，和项目无关，不需要管它。
templates           #项目模板文件目录，用来存放模板文件
manage.py     #命令行工具，通过可以与项目与行交互。在终端输入python manege.py help，可以查看功能。
```

### **通过命令行，添加新的APP**

> 首先打开Pycharm内建的Terminal

<img src="/img/Xnip2019-12-08_20-51-57.png" style="zoom:50%;" />

输入`python manage.py startapp bbs`创建名为bbs的APP

<img src="/img/Xnip2019-12-08_20-54-46.png" style="zoom:50%;" />

> 创建成功会出现`bbs`文件夹

关于更多的一些Django常用的命令，大家可以看看这篇文章：[Django常用命令](https://www.django.cn/forum/forum-5.html)

### 迁移数据库

在Terminal下输入下面的命令，生成和同步数据库：

```sh
python manage.py makemigrations
python manage.py migrate
```

![](/img/Xnip2019-12-08_20-50-36.png)

### **启动Django项目**

- 在Terminal下输入

```
python manage.py runserver 8080
```

8080是我们指定的启动端口，如果不指定，默认则是`8000`

- Pycharm直接运行

点击右上角的▶️图标

<img src="/img/Xnip2019-12-08_21-02-48.png" style="zoom:50%;" />

> 运行效果：

![](/img/Xnip2019-12-08_21-00-41.png)



这样，我们的第一个Django项目就创建完成了。