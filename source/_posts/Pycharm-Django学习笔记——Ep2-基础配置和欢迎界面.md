---
title: Pycharm-Django学习笔记——Ep2-基础配置和欢迎界面
date: 2019-12-13 09:51:35
index_img: /img/Xnip2019-12-08_21-35-43.png
banner_img: /img/Xnip2019-12-08_21-35-43.png
tags: [Python, Django]
categories: 代码
---

### 基础配置

创建项目之后，我们需要对项目进行最基础的配置。这些配置是我们做项目的时候必须要配置的，所以我们先提前配置好。

<!-- more -->

我们打开myblog目录下的settings.py文件。

##### 一、设置域名访问权限

```python
myblog/settings.py
ALLOWED_HOSTS = []      #修改前
ALLOWED_HOSTS = ['*']   #修改后，表示任何域名都能访问。如果指定域名的话，在''里放入指定的域名即可
```

##### 二、设置TEMPLATES里的'DIRS'，添加模板目录templates的路径，后面我们做网站模板的时候用得着。

```python
myblog/settings.py
#修改前
'DIRS': []
#修改后
'DIRS': [os.path.join(BASE_DIR, 'templates')]
注：使用pycharm创建的话会自动添加
```

##### 三、找到DATABASES设置网站数据库类型。

这里我们使用默认的sqlite3。如果需要使用Mysql请查看文章：[Django如何使用Mysql数据库](https://www.django.cn/forum/forum-6.html)，其它数据库请查看官方文档。[官方文档](https://docs.djangoproject.com/en/2.1/ref/settings/#databases)，后期上线部署的话，也可以进行数据库与数据库之间的数据转换。具体可查看：[如何把SQLite数据库转换为Mysql数据库](https://www.django.cn/article/show-17.html)

##### 四、在INSTALLED_APPS添加APP应用名称。

```python
myblog/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    ....
    'blog.apps.BlogConfig',#注册APP应用
]
#使用pycharm创建的话，这里自动添加了，如果是终端命令创建的话，需要手动添加应用名称如'blog',
```

##### 五、修改项目语言和时区

```python
myblog/settings.py
#修改前为英文
LANGUAGE_CODE = 'en-us'
#修改后
LANGUAGE_CODE = 'zh-hans' #语言修改为中文
#时区，修改前
TIME_ZONE = 'UTC'
#修改后
TIME_ZONE = 'Asia/Shanghai' #
```

##### 六、在项目根目录里创建static和media，两个目录。

static用来存放模板CSS、JS、图片等静态资源，media用来存放上传的文件，后面我们在讲解数据库创建的时候有说明。

settings里找到STATIC_URL，然后在后面一行加上如下代码。

```python
myblog/settings.py

#设置静态文件目录和名称
STATIC_URL = '/static/'

#加入下面代码

#这个是设置静态文件夹目录的路径
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static'),
)
#设置文件上传路径，图片上传、文件上传都会存放在此目录里
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

基本配置完成，更多关于配置文件的介绍，请查看文章：[全局配置settings详解](https://www.django.cn/course/show-10.html)

### 欢迎界面

基础配置做好了之后，我们就可以先迁移数据到数据库，然后启动我们的项目，感受Django的魅力。

在Pycharm左下角底部的Terminal，会弹出Terminal终端窗口，Pycharm自动会帮我们启动虚拟环境。如下图所示：

![3.jpg](https://www.django.cn/media/upimg/3_20181009212347_421.jpg)

这里面有两个地方需要留意：

1、留意项目路径，看这个路径是不是我们项目的路径。

2、留意路径前有没有我们创建的虚拟环境名，之前我们创建的虚拟环境名是myblogvenv，如果显示正确，则说明我们启动正确。如果没有虚拟环境名，则进入项目目录下的myblogvenv\Scripts目录里，在终端输入activate启动虚拟环境，然后再切换到项目根目录里。如果前面的虚拟环境名称不对，则在终端输入deactivate退出虚拟环境，然后按上面的方法启动虚拟环境。

上面都OK了，我们就在终端里依次输入如下命令进行数据库迁移：

```
python manage.py makemigrations
python manage.py migrate
```

![4.jpg](https://www.django.cn/media/upimg/4_20181009213423_897.jpg)

迁移数据之后，网站目录里自动会创建一个数据库文件db.sqlite3，里面存放着我们的数据。

![14.jpg](https://www.django.cn/media/upimg/14_20181010001746_429.jpg)

之后输入下面命令创建管理帐号和密码：

```
python manage.py createsuperuser
```

![5.jpg](https://www.django.cn/media/upimg/5_20181009213649_680.jpg)

注意：密码不要太简单或者和电子邮件相似，不然Django会有风险提示。

最后，我们输入下面其中一个命令，启动我们的Django项目：

```sh
python manage.py runserver #默认使用8000端口
python manage.py runserver 8080 #指定启动端口
python manage.py runserver 127.0.0.1:9000 #指定IP和端口
```

![6.jpg](https://www.django.cn/media/upimg/6_20181009214014_442.jpg)

提示启动成功，然后我们在浏览器里输入：http://127.0.0.1:8000/

就可以查看到Django默认的欢迎页面！

![7.jpg](https://www.django.cn/media/upimg/7_20181009214331_427.jpg)

是不是有一种成就感？这就是Django的强大之处。几个命令就可以实现一个网站创建。自己动手试试吧。

关于更多的Django命令，大家可以查看文章：[Django常用命令](https://www.django.cn/course/show-4.html)

有的朋友觉得这还是不过瘾，说这个欢迎页面是Django自带的，我们能自己做一个欢迎页面么？答案是肯定的。

首先，打开打开bolg目录下的views.py文件，在里面输入：

```python
myblog/blog/views.py

from django.http import HttpResponse

def hello(request):
    return HttpResponse('欢迎使用Django！')
```

再打开myblog目录下的urls.py文件，在文件里添加两行代码：

```python
myblog/myblog/urls.py

from django.contrib import admin
from django.urls import path
from blog import views         #+ 
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.hello),   #+
]
```

留意：代码后面有个**#+**表示是我新添加的代码。

代码写好之后，启动项目，刷新页面。就可以看到：

![8.jpg](https://www.django.cn/media/upimg/8_20181009215736_355.jpg)

OK，自定义欢迎页面成功显示！

之后，我们在浏览器里面访问：http://127.0.0.1:8000/admin 就可以进入Django自带的后台管理。

![9.jpg](https://www.django.cn/media/upimg/9_20181009220344_400.jpg)

输入刚才我们创建的帐号与密码，点击登录。

![10.jpg](https://www.django.cn/media/upimg/10_20181009220449_945.jpg)

进入到管理后台，这个后台功能十分强大。后面我们会对其进行详细介绍。



---

此教程来自 [Django中文网](https://www.django.cn/course/course-2.html)