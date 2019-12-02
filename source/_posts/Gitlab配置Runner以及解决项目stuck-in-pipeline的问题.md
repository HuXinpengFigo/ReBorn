---
title: Gitlab配置Runner以及解决项目stuck in pipeline的问题
date: 2019-11-25 12:21:55
index_img: /img/Xnip2019-11-25_12-37-06.png
banner_img: /img/292DD11A-C1A1-473D-8628-2F16D39F4007.png
tags: [Git,版本控制]
categories: 代码
---

> 在push项目到Gitlab之后，出现错误，错误信息为：This job is stuck, because you don’t have any active runners that can run this job
>
> 则需要配置好Runner了

<!-- more -->

#### gitlab-ci runner的安装与配置（以Mac为例）

runner可以理解为一个环境，相当于jenkins的slave，机器（或者是docker），通过 runner程序与git服务器进行通信，当有新的任务发布到runner时，runner会执行.gitlab-ci.yml所定义的ci指令。

runner有三种模式， sharedRunner，specific runners和 group runners。gitlab上可以使用官方的shared runners，创建runner需要git管理员的权限。

本次配置采用手动配置specific runner。

### 安装Runner

#### 1、使用homebrew安装

命令行输入：

```sh
brew install gitlab-runner
```

#### 2、官方安装

Gitlab runner 10以上安装方式，若安装旧版本前往官网查看

##### 下载：

```sh
sudo curl --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64
```

##### 设置权限：

```sh
sudo chmod +x /usr/local/bin/gitlab-runner
```

### 注册Runner

注册runner，你需要有一个项目，并且需要至少master权限。
打开**settings->CI/CD**页面，选择第二项**Runners settings**，左侧会显示与当前项目相关的参数。

![](/Users/figo/截图/Xnip2019-11-25_09-54-51.png)

##### 执行

```sh
gitlab-runner register
```

##### 指定git的URL

```sh
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
https://git.lug.ustc.edu.cn/
```

##### 指定gitlab-runner的token

```sh
Please enter the gitlab-ci token for this runner:
<<your token>>
```

> 上图的第二个红线所指的地方

##### 给CI的描述

```sh
Please enter the gitlab-ci description for this runner:
[huxinpengdeMacBook-Pro.local]: blog
```

##### 关联git和runner的tag

```sh
Please enter the gitlab-ci tags for this runner (comma separated):
pages，test<your tag>
```

##### 选择runner的执行环境

```sh
Please enter the executor: custom, shell, virtualbox, docker+machine, docker, docker-ssh, parallels, ssh, docker-ssh+machine, kubernetes:
shell<mac上可以直接使用shell>

```

#### 若选择docker，则需要下一步

##### 指定docker的image

```sh
 Please enter the Docker image (eg. ruby:2.1):
 alpine:latest

```

> * url：私有git的路径
>
> * token：项目的token，用于关联runner和项目
>
> * name：runner的名字，用于区分runner
>
> * tags：用于匹配任务（jobs）和执行任务的设备（runners）
>
> * executor：执行环境

当我们完成设置后，可通过`vi ~/.gitlab-runner/config.toml`打开runner 的配置文件看到之前配置的内容。

### 启动

```sh
cd ~
gitlab-runner install
gitlab-runner start
或者 brew services start gitlab-runner

```

当所有步骤执行后，在**Runners settings**会显示runner的状态，显示为绿色，则runner配置成功。

----

参考：

[1] [gitlab-runner的配置——for Mac](https://www.jianshu.com/p/c7995ad64f48)
[2] [gitlab-runner官方文档