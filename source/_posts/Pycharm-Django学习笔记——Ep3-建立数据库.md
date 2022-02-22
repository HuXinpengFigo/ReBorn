---
title: Pycharm-Django学习笔记——Ep3-建立数据库
date: 2019-12-19 14:19:38
index_img: /img/1200px-SQLite370.svg.png
banner_img: /img/1200px-SQLite370.svg.png
tags: [Python,Django,Sqlite]
categories: 代码
---

### 数据库分析

从网站需求分析及网站功能、页面设计可以知道，我们的Blog主要以文章内容为主。所以我们在设计数据库的时候，我们主要以文章信息为核心数据，然后逐步向外扩展相关联的数据信息。

<!-- more -->

从这篇文章http://demo.django.cn/show-10.html可以看到，文章有标题、分类、作者、浏览次数、发布时间、文章标签等信息。

![1111.jpg](https://www.django.cn/media/upimg/1111_20181009173401_998.jpg)

![2222.jpg](https://www.django.cn/media/upimg/2222_20181009173414_583.jpg)

这其中，**文章与分类**的关系是一对多的关系，什么是一对多？就是一篇文章只能有一个分类，而一个分类里可以有多篇文章。**文章与标签**的关系是多对多的关系，多对多简单理解就是，一篇文章可以有多个标签，一个标签里同样可以有多篇文章。关于一对多、多对多，大家可以查看这篇文章：[多个数据模型间的关系](https://www.django.cn/course/show-13.html)

我们将文章表命名为Article，通过前面的分析得出文章信息表Article的数据库结构如下：

| **表字段**  | **字段类型**            | **备注**                              |
| ------------ | ------------------------ | -------------------------------------- |
| id           | int类型，长度为11        | 主键，由系统自动生成                   |
| title        | CharField类型，长度为100 | 文章标题                               |
| category     | ForeignKey               | 外键，关联文章分类表                   |
| tags         | ManyToManyField          | 多对多，关联标签列表                   |
| body         | TextField                | 文章内容                               |
| user         | ForeignKey               | 外键，文章作者关联用户模型，系统自带的 |
| views        | PositiveIntegerField     | 文章浏览数，正的整数，不能为负         |
| tui          | ForeignKey               | 外键，关联推荐位表                     |
| created_time | DateTimeField            | 文章发布时间                           |

从文章表里，我们关联了一个**分类表**，我们把这个分类表命名为category，category表的数据库结构如下：

| **表字段** | **字段类型**            | **备注**       |
| ---------- | ----------------------- | -------------------- |
| id         | int类型，长度为11       | 主键，由系统自动生成 |
| name       | CharField类型，长度为30 | 分类名               |

文章关联的**标签表**，我们命名为tag，结构如下：

| **表字段** | **字段类型**            | **备注**       |
| ---------- | ----------------------- | -------------------- |
| id         | int类型，长度为11       | 主键，由系统自动生成 |
| name       | CharField类型，长度为30 | 标签名               |

文章关联的**推荐位表**，命名为tui，结构如下：

| **表字段** | **字段类型**            | **备注**       |
| ---------- | ----------------------- | -------------------- |
| id         | int类型，长度为11       | 主键，由系统自动生成 |
| name       | CharField类型，长度为30 | 标签名               |

除此之外，我们还有两个独立的表，和文章没有关联的，一个是幻灯图片的表，一个是友情链接的表。

**幻灯图表**，命名为banner，数据库结构如下：

| **表字段** | **字段类型**             | **备注**                             |
| ---------- | ------------------------ | ------------------------------------ |
| id         | int类型，长度为11        | 主键，由系统自动生成                 |
| text_info  | CharField类型，长度为100 | 标题，图片文本信息                   |
| img        | ImageField类型           | 图片类型，保存传图片的路径           |
| link_url   | URLField类型             | 图片链接的URL                        |
| is_active  | BooleanField布尔类型     | 有True 和False两个值，意思为是否激活 |

**友情链接**表命名为link，结构如下：

| **表字段** | **字段类型**            | **备注**             |
| ---------- | ----------------------- | -------------------- |
| id         | int类型，长度为11       | 主键，由系统自动生成 |
| name       | CharField类型，长度为70 | 友情链接的名称       |
| linkurl    | URLField类型            | 友情链接的URL        |

至此，我们的数据库构造大致完成，后期如果还有其它的需求，我们可以在这基础上进行增加或者删除。下面我们就开始进行项目的创建与开发。

# 创建数据库模型

Django是通过Model操作数据库，不管你数据库的类型是MySql或者Sqlite，Django它自动帮你生成相应数据库类型的SQL语句，所以不需要关注SQL语句和类型，对数据的操作Django帮我们自动完成。只要回写Model就可以了！

django根据代码中定义的类来自动生成数据库表。我们写的类表示数据库的表，如果根据这个类创建的对象是数据库表里的一行数据，对象.id 对象.value是每一行里的数据。

基本的原则如下：
每个模型在Django中的存在形式为一个Python类
每个模型都是django.db.models.Model的子类
模型里的每个类代表数据库中的一个表
模型的每个字段（属性）代表数据表的某一列
Django将自动为你生成数据库访问API

之前我们在前面的[数据库设计分析](https://www.django.cn/course/show-34.html)文章里已经分析过数据库的结构。完成博客，我们需要存储六种数据：文章分类、文章、文章标签、幻灯图、推荐位、友情链接。每种数据一个表。

**分类表结构设计**：

表名：Category、分类名：name

**标签表设计：**

表名：Tag、标签名：name

**文章表结构设计：**

表名：Article、标题：title、摘要：excerpt、分类：category、标签：tags、推荐位、内容：body、创建时间：created_time、作者：user、文章封面图片img

**幻灯图表结构设计：**

表名：Banner、图片文本text_info、图片img、图片链接link_url、图片状态is_active。

**推荐位表结构设计：**

表名：Tui、推荐位名name。

**友情链接表结构设计：**

表名：Link、链接名name、链接网址linkurl。

**其中：**

文章和分类是一对多的关系，文章和标签是多对多的关系，文章和作者是一对多的关系，文章和推荐位是一对多关系(看自己的需求，也可以设计成多对多)。

打开blog/models.py,输入代码：

```python
from django.contrib.auth.models import User
from django.db import models


# 导入Django自带用户模块

# 文章分类
class Category(models.Model):
    name = models.CharField('博客分类', max_length=100)
    index = models.IntegerField(default=999, verbose_name='分类排序')

    class Meta:
        verbose_name = '博客分类'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.name


# 文章标签
class Tag(models.Model):
    name = models.CharField('文章标签', max_length=100)

    class Meta:
        verbose_name = '文章标签'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.name


# 推荐位
class Tui(models.Model):
    name = models.CharField('推荐位', max_length=100)

    class Meta:
        verbose_name = '推荐位'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.name


# 文章
class Article(models.Model):
    title = models.CharField('标题', max_length=70)
    excerpt = models.TextField('摘要', max_length=200, blank=True)
    category = models.ForeignKey(Category, on_delete=models.DO_NOTHING, verbose_name='分类', blank=True, null=True)
    # 使用外键关联分类表与分类是一对多关系
    tags = models.ManyToManyField(Tag, verbose_name='标签', blank=True)
    # 使用外键关联标签表与标签是多对多关系
    img = models.ImageField(upload_to='article_img/%Y/%m/%d/', verbose_name='文章图片', blank=True, null=True)
    body = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name='作者')


    user = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name='作者', null=True)
    views = models.PositiveIntegerField('阅读量', default=0)
    tui = models.ForeignKey(Tui, on_delete=models.DO_NOTHING, verbose_name='推荐位', blank=True, null=True)

    created_time = models.DateTimeField('发布时间', auto_now_add=True, null=True)
    modified_time = models.DateTimeField('修改时间', auto_now=True)


    class Meta:
        verbose_name = '文章'
        verbose_name_plural = '文章'


    def __str__(self):
        return self.title


# Banner
class Banner(models.Model):
    text_info = models.CharField('标题', max_length=50, default='')
    img = models.ImageField('轮播图', upload_to='banner/')
    link_url = models.URLField('图片链接', max_length=100)
    is_active = models.BooleanField('是否是active', default=False)

    def __str__(self):
        return self.text_info

    class Meta:
        verbose_name = '轮播图'
        verbose_name_plural = '轮播图'


# 友情链接
class Link(models.Model):
    name = models.CharField('链接名称', max_length=20)
    linkurl = models.URLField('网址', max_length=100)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name = '友情链接'
        verbose_name_plural = '友情链接'
```

这里面我们多增加了一个img图片封面字段，用于上传文章封面图片的，article_img/为上传目录，%Y/%m/%d/为自动在上传的图片上加上文件上传的时间。

里面的模型字段与模型元数据Meta选项详解我在这里就不做过多介绍，更多请点击文章[数据模型字段及属性详解](https://www.django.cn/course/show-12.html)和[模型元数据Meta选项详解](https://www.django.cn/course/show-14.html)了解。

我们已经编写了博客数据库模型的代码，但那还只是 Python 代码而已，Django 还没有把它翻译成数据库语言，因此实际上这些数据库表还没有真正的在数据库中创建。我们需要进行数据库迁移。

在迁移之前，我们先需要设置数据库，如果我们使用默认的数据库的话，就不需要设置，Django默认使用

sqlite3数据库，如果我们想使用Mysql数据库的话，则需要我们单独配置。我们打开settings.py文件，找到DATABASES，然后把它修改成如下代码：

```python
############修改成mysql如下：
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'test',    #你的数据库名称
        'USER': 'root',   #你的数据库用户名
        'PASSWORD': '445813', #你的数据库密码
        'HOST': '', #你的数据库主机，留空默认为localhost
        'PORT': '3306', #你的数据库端口
    }}

#由于mysql默认引擎为MySQLdb，在__init__.py文件中添加下面代码
#在python3中须替换为pymysql,可在主配置文件（和项目同名的文件下，不是app配置文件）中增加如下代码
#import pymysql
#pymysql.install_as_MySQLdb()
#如果找不到pymysql板块，则通过pip install pymysql进行安装。
```

更多关于Django数据库的配置，请查看官方文档：[数据库设置](https://docs.djangoproject.com/en/2.1/ref/settings/#databases)

数据库设置好之后，我们就依次输入下面的命令进行数据库迁移：

```sh
python manage.py makemigrations
python manage.py migrate
```

迁移的时候，会有如下提示：

![11.jpg](https://www.django.cn/media/upimg/11_20181010000432_323.jpg)

出现这个原因是因为我们的幻灯图使用到图片字段，我们需要引入图片处理包。提示里也给了我们处理方案，输入如下命令，安装Pillow模块即可：

```sh
pip install Pillow
```

![12.jpg](https://www.django.cn/media/upimg/12_20181010000724_924.jpg)

安装成功之后再迁移数据库

![13.jpg](https://www.django.cn/media/upimg/13_20181010000839_542.jpg)

数据库迁移成功之后，程序会在blog下的migrations目录里自动生成几个000开头的文件，文件里面记录着数据库迁移记录：

![1.jpg](https://www.django.cn/media/upimg/1_20181010005709_849.jpg)

大家可以查看一下。了解迁移的过程。本文就不做过多介绍。

### 用Admin管理后台管理数据

上节我们我们把数据库迁移到数据库里去了，那么现在我们数据库里是个什么样的情况呢？我们点击Pycharm右上角的Database，然后在网站项目里选中我们的数据库文件db.sqlite3，把它拖到Database框里。

![15.jpg](https://www.django.cn/media/upimg/15_20181010002045_529.jpg)

然后点击db，就可以查看到我们的网站数据库，我们可以对数据进行增、删、改、查操作。

![16.jpg](https://www.django.cn/media/upimg/16_20181010002353_211.jpg)

更多相关方面的操作请查看文章：[使用Pycharm里的Database对数据库进行可视化操作](https://www.django.cn/article/show-13.html)

Pycharm Batabase限制非常大，下面我们介绍如何使用Django自带的admin管理网站数据。django的admin后台管理它可以让我们快速便捷管理数据，我们可以在各个app目录下的admin.py文件中对其进行控制。想要对APP应用进行管理，最基本的前提是要先在settings里对其进行注册，就是在INSTALLED_APPS里把APP名添加进去，我们在前面的文章[基础配置](https://www.django.cn/course/show-36.html)有提到过。

注册APP应用之后，我们想要在admin后台里对数据库表进行操作，我们还得在应用APP下的admin.py文件里对数据库表先进行注册。我们的APP应用是blog，所以我们需要在blog/admin.py文件里进行注册：

```python
blog/admin.py

from django.contrib import admin

from .models import Banner, Category, Tag, Tui, Article, Link
#导入需要管理的数据库表

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ('id', 'category', 'title', 'tui', 'user', 'views', 'created_time')
    # 文章列表里显示想要显示的字段
    list_per_page = 50
    # 满50条数据就自动分页
    ordering = ('-created_time',)
    # 后台数据列表排序方式
    list_display_links = ('id', 'title')
    # 设置哪些字段可以点击进入编辑界面



@admin.register(Banner)
class BannerAdmin(admin.ModelAdmin):
    list_display = ('id', 'text_info', 'img', 'link_url', 'is_active')

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ('id', 'name', 'index')

@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    list_display = ('id', 'name')

@admin.register(Tui)
class TuiAdmin(admin.ModelAdmin):
    list_display = ('id', 'name')

@admin.register(Link)
class LinkAdmin(admin.ModelAdmin):
    list_display = ('id', 'name','linkurl')
```

关于admin定制和数据库表注册管理方法，在文章[定制Admin管理后台](https://www.django.cn/course/show-16.html)有详细介绍。

登录管理后台http://127.0.0.1:8000/admin/

注册之前的后台：

![17.jpg](https://www.django.cn/media/upimg/17_20181010004347_213.jpg)

注册之后，启动项目，刷新页面：

![18.jpg](https://www.django.cn/media/upimg/18_20181010005104_145.jpg)

多出了之前我们在models里创建的表。我们可以在后台里面对这些表进行增、删、改方面的操作。

### 使用富文本编辑器添加数据

在Django admin后台添加数据的时候，文章内容文本框想发布一篇图文并茂的文章需就得手写Html代码，这十分吃力，也没法上传图片和文件。这显然不是我等高大上程序猿想要的。

![2.jpg](https://www.django.cn/media/upimg/2_20181010013509_103.jpg)

为提升效率，我们可以使用富文本编辑器添加数据。支持Django的富文本编辑器很多，这里我推荐使用DjangoUeditor，Ueditor是百度开发的一个富文本编辑器，功能强大。下面教大家安装如何使用DjangoUeditor。

1、首先我们先下载DjangoUeditor包。点击下面的链接进行下载！下载完成然后解压到项目根目录里。

 [DjangoUeditor.zip](https://www.django.cn/media/upfile/DjangoUeditor_20181010013851_248.zip)

2、settings.py里注册APP，在INSTALLED_APPS里添加'DjangoUeditor',。

```python
myblog/settings.y
INSTALLED_APPS = [
    'django.contrib.admin',
    ....
    'DjangoUeditor', #注册APP应用
]
```

3、myblog/urls.py里添加url。

```python
myblog/urls.py
...
from django.urls import path, include
#留意上面这行比原来多了一个include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.hello),
    path('ueditor/', include('DjangoUeditor.urls')), #添加DjangoUeditor的URL
]
```

4、修改blog/models.py里需要使用富文本编辑器渲染的字段。这里面我们要修改的是Article表里的body字段。

把原来的：

```python
blog/models.py

body = models.TextField()
```

修改成：

```python
blog/models.py
from DjangoUeditor.models import UEditorField #头部增加这行代码导入UEditorField

body = UEditorField('内容', width=800, height=500, 
                    toolbars="full", imagePath="upimg/", filePath="upfile/",
                    upload_settings={"imageMaxSize": 1204000},
                    settings={}, command=None, blank=True
                    )
```

留意里面的imagePath="upimg/", filePath="upfile/" 这两个是图片和文件上传的路径，我们上传文件，会自动上传到项目根目录media文件夹下对应的upimg和upfile目录里，这个目录名可以自行定义。有的人问，为什么会上传到media目录里去呢？那是因为之前我们在[基础配置](https://www.django.cn/course/show-36.html)文章里，设置了上传文件目录media。

上面步骤完成后，我们启动项目，进入文章发布页面。提示出错：

```
render() got an unexpected keyword argument 'renderer'
```

![3.jpg](https://www.django.cn/media/upimg/3_20181010020114_464.jpg)

错误页面上有提示，出错的地方是下面文件的93行。

```
F:\course\myblog\myblogvenv\lib\site-packages\django\forms\boundfield.py in as_widget, line 93
```

我这里使用的是最新版本的Django2.1.1所以报错，解决办法很简单。打开这个文件的93行，注释这行即可。

![4.jpg](https://www.django.cn/media/upimg/4_20181010020705_975.jpg)

修改成之后，重新刷新页面，就可以看到我们的富文本编辑器正常显示。

![5.jpg](https://www.django.cn/media/upimg/5_20181010020936_275.jpg)

留意，如果我们在富文本编辑器里，上传图片，在编辑器内容里不显示上传的图片。那我们还需要进行如下设置，打开myblog/urls.py文件，在里面输入如下代码：

```python
myblog/urls.py
....
from django.contrib import admin
from django.urls import path, include, re_path
# 上面这行多加了一个re_path
from django.views.static import serve
# 导入静态文件模块
from myblog import settings
# 导入配置文件里的文件上传配置
from blog import views         # +

urlpatterns = [
    path('admin/', admin.site.urls),
    ....
    re_path('^media/(?P<path>.*)$', serve, {'document_root': settings.MEDIA_ROOT}),#增加此行
]
```

设置好了之后，图片就会正常显示。这样我们就可以用DjangoUeditor富文本编辑器发布图文并茂的文章了。

----

此教程来自 [Django中文网](https://www.django.cn/course/course-2.html)