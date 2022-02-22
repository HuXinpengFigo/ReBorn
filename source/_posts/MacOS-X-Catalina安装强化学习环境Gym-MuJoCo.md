---
title: MacOS X Catalina安装强化学习环境Gym+MuJoCo
date: 2020-02-26 17:50:42
index_img: /img/kissclipart-logo-de-mac-os-clipart-macos-computer-icons-81d21f0ecb21adb8.png
banner_img: /img/m8c20gchf7231.jpg
tags: [MacOS,强化学习]
categories: MacOS,Gym,MuJoCo
---

# MacOS X Catalina安装强化学习环境Gym+MuJoCo

### 最终配置：

> Python vision -- 3.7.2
>
> Gym version -- 0.17.0
>
> MuJoCo v200
>
> Mujoco-py  2.0.18
>
> MacOS X 10.15 Catalina

<!-- more -->

### 安装：

#### 1）获取MuJoCo的License

在MuJoCo官网注册并下载获取电脑id的文件**getid_osx**，赋予其可执行权限

```sh
chmod +x getid_linux
./getid_linux
```

之后按照官网的提示来激活，之后会得到OpenAI发来的License:<***mjkey.txt***>

### 2）下载mujoco200_macos.zip

### 3）解压mujoco200_macos.zip到隐藏文件夹.mujoco中（自行创建）

```sh
unzip mujoco200_macos.zip -d ~/.mujoco
```

.mujoco必须要创建在根目录

> 建议把mujoco200_macos文件夹里面的东西直接复制出来放到~/.mujoco/mujoco200文件夹中，以避免跑sample的时候报错

### 4）下载*mjkey.txt*并放到*.mujoco*中

### 5）vim ~/.bashrc

### 6）复制以下规则到你的bashrc文件中

```sh
export LD_LIBRARY_PATH=~/.mujoco/mujoco200/bin${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export MUJOCO_KEY_PATH=~/.mujoco/mujoco200/${MUJOCO_KEY_PATH}
```

### 7）source .bashrc

### 8）在git上clone下mujoco-py项目

也可以用pip来安装，但是我用pip安装的有问题

```sh
git clone https://github.com/openai/mujoco-py.git
```

### 9）之后安装各种依赖

##### For MacOS:

```sh
cd mujoco-py/
brew install patchelf
brew install python3 python-dev python3-dev build-essential libssl-dev libffi-dev libxml2-dev libxslt1-dev zlib1g-dev libglew1.5 libglew-dev python-pip
brew install libgl1-mesa-dev libgl1-mesa-glx libosmesa6-dev python3-pip python3-numpy python3-scipy 
sudo pip3 install -r requirements.txt
sudo pip3 install -r requirements.dev.txt
sudo python3 setup.py install
sudo pip3 install gym
```

由于MacOS版本是由Ubuntu版本改来的，其中安装的部分依赖可能安装失败

如果pip安装时间过长可能会报错超时，在安装命令后面加上延时就行：

```sh
pip3 install gym --default-timeout=1000
```

##### For Ubuntu:

```sh
cd mujoco-py/
sudo apt-get update
sudo apt-get install patchelf
sudo apt-get install python3 python-dev python3-dev build-essential libssl-dev libffi-dev libxml2-dev libxslt1-dev zlib1g-dev libglew1.5 libglew-dev python-pip
sudo apt-get install libgl1-mesa-dev libgl1-mesa-glx libosmesa6-dev python3-pip python3-numpy python3-scipy 
sudo pip3 install -r requirements.txt
sudo pip3 install -r requirements.dev.txt
sudo python3 setup.py install
sudo pip3 install gym
```

**PS:**你需要每次都在~/.mujoco-py文件夹中来跑仿真。如果想要在任意链接跑的话就把上述

***sudo python3 setup.py install***换成***sudo python3 setup.py develop***

至此我们已经安装完了，接下来就是测试是否安装成功了。

### 10）测试

```python
import gym
env = gym.make('Humanoid-v2')
env.reset()
for _ in range(1000):
  env.render()
  env.step(env.action_space.sample()) # take a random action
```

![image-20200226174457818](/img/image-20200226174457818.png)

**Note:**如果你需要Cuda和nvidia的驱动在你的Linux上，那有可能会出现关于openGL的问题。可以试试在bash中添加以下规则：

```sh
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libGLEW.so:/usr/lib/nvidia-384/libGL.so
```

