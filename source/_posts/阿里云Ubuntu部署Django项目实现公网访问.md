---
title: 阿里云Ubuntu部署Django项目实现公网访问
date: 2019-12-19 11:55:35
index_img: /img/Xnip2019-12-19_12-00-29.png
banner_img: /img/Xnip2019-12-19_12-00-29.png
tags: [Python, Django]
categories: 配置
---

### 购买服务器

首先买好阿里云的服务器，这点不用多言。

刚开始选择默认系统时可以随意选择，在之后可以更换。

<!-- more -->

> 刚开始可选择的系统版本很少，而且默认的Ubuntu只有16.04，且默认安装Python 3.5，后期调整很麻烦。

最终我们更换为Ubuntu 18.04（该系统默认安装Python 3.6，可以一定程度上省去对Python环境的配置）

### 进入控制台

`已开通的云产品`->`云服务器ECS`进入当前购买的服务器管理控制台

![](/img/Xnip2019-12-19_11-21-03.png)

`实例`->`远程连接`进入该服务器的控制台

![](/img/Xnip2019-12-19_11-22-50.png)

设置好界面之后进入控制台。

### 配置环境

> 由于项目是由Python3+Django+sqlite3组成，我们需要安装的依次为：
>
> 1. pip3
> 2. Django
> 3. sqlite3
> 4. virtualenv
> 5. Git

#### Pip3

> 此处需要注意的是阿里云服务器刚打开直接使用`apt-get install python3-pip`会提示无法定位到软件包，我们需要先进行：
>
> ```shell
> apt update
> apt upgrade
> ```
>
> 再进行：
>
> ```shell
> apt-get install python3-pip
> ```

#### Django

```shell
pip3 install django
```

#### Sqlite3

```sh
apt-get install sqlite3
```

#### virtualenv

安装virtualenv

```sh
pip3 install virtualenv
pip3 install virtualenvwrapper #安装虚拟环境管理工具
```

在home下创建虚拟环境安装目录

```sh
mkdir .virtualenvs
```

为virtualenv配置环境变量,打开.bashrc文件，在末尾加上两行代码，在阿里云的ubuntu上，你想编辑文件只能用vi/vim打开，对于没用过vi的话还是需要点时间学习的。或者你可以在本地pc编辑好，再用Xftp工具上传覆盖原来的文件。

用vim打开.bashrc ，一般就在home文件夹下

```sh
sudo vim ~/.bashrc
```

在末尾添加两行代码

```sh
export WORKON_HOME=$HOME/.virtualenvs  # 所有虚拟环境存储的目录
source /usr/local/bin/virtualenvwrapper.sh
```

使配置文件生效

```sh
source ~/.bashrc

```

#### Git

```sh
apt install git

```

### 运行项目

首先将项目拉取到本地：

```sh
git clone <你的项目地址>

```

进入项目根目录之后：

```sh
python manage.py makemigrations
python manage.py migrate
python manage.py runserver 0.0.0.0:8080

```

### 设置公网访问

`实例列表`->`更多`->`网络和安全组`->`安全组配置`->`配置规则`

<img src="/img/Xnip2019-12-19_11-43-28.png" style="zoom:50%;" />

到达下界面后点击`添加安全组规则`

![](/img/Xnip2019-12-19_11-45-26.png)

按下图配置好

<img src="/img/Xnip2019-12-19_11-48-53.png" style="zoom:50%;" />

还有一步，在你的Django项目中`setting.py`中设置：

> - INSTALLED_APPS中应用的添加
>
> - ALLOWED_HOSTS主机的设置
>
> 　　　方式1：　
>
> ```py
> ALLOWED_HOSTS = ['外网ip','localhost', '0.0.0.0:8000', '127.0.0.1',]
> 
> ```
>
> 　　　方式2：
>
> ```sh
> ALLOWED_HOSTS = ['*']
> 
> ```

以上都配置好之后重启一下你的Django项目。

----

全部配置好之后就可以在`<你的公网IP>:8000`这个地址访问你的项目了！

