---
title: Docker入门
date: 2020-03-31 16:40:47
index_img: /img/docker-1.png
banner_img: /img/docker-cloud-twitter-card.png
tags: [docker]
categories: 代码
---

### 检查docker版本代码

<!-- more -->

```sh
docker --version

Docker version 17.12.0-ce, build c97c6d6
```

### 查看全局docker信息

```sh
docker info

Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.12.0-ce
Storage Driver: overlay2
...
```

### 查看当前所有Image

```sh
docker image ls
```

### 运行Image

```sh
docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

### 查看所有Container

```sh
docker container ls --all

CONTAINER ID     IMAGE           COMMAND      CREATED            STATUS
54f4984ed6a8     hello-world     "/hello"     20 seconds ago     Exited (0) 19 seconds ago
```

### 命令总结

```sh
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```

## Define a container with `Dockerfile`

首先，创建一个空文件夹来存放`Dockerfile`，创建的`Dockerfile`内容如下：

```sh
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

再多创建两个文件`requirements.txt`&`app.py`:

" When the above `Dockerfile` is built into an image, `app.py` and `requirements.txt` is present because of that `Dockerfile`’s `COPY` command, and the output from `app.py` is accessible over HTTP thanks to the `EXPOSE` command."

可以利用 `ADD` 命令复制本地文件到镜像；用 `EXPOSE` 命令来向外部开放端口；用 `CMD` 命令来描述容器启动后运行的程序

### `requirements.txt`

```sh
Flask
Redis
```

### `app.py`

```sh
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

你不需要在自己电脑里安装`Flask`和`Redis`使用`requirements.txt`就可以往镜像中安装以上库。

## 构建该镜像

```sh
$ ls
Dockerfile		app.py			requirements.txt
```

以下命令能够构建该镜像，用`--tag`选项为该镜像命名，用`-t`可以代替。

```sh
docker build --tag=friendlyhello .
```

镜像构建好之后，用以下命令可以看到它加入了本地的Image仓库中。

```sh
$ docker image ls

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```

现在`TAG`默认为`latest`，完整的语法为`--tag=friendlyhello:v0.0.1`。

### 运行该镜像

```sh
docker run -p 4000:80 friendlyhello
```

由返回的信息可知，Python正在`http://0.0.0.0:80`为该镜像提供服务，但是该信息是由容器内部发出的，在本机中端口从80被映射到了4000，在浏览器中输入`http://localhost:4000`可看到正确的信息。

![local](/home/figo/桌面/Docker入门/local.png)

你可以用`curl`命令在终端中看到相同的内容：

```sh
$ curl http://localhost:4000

<h3>Hello World!</h3><b>Hostname:</b> 8fc990912a14<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```

在终端中按`CTRL+C`来退出。

现在将该镜像在后台、分离模式下运行：

```sh
docker run -d -p 4000:80 friendlyhello
```

此时查看信息变得很重要：

```sh
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago
```

关闭后台运行的容器：

```sh
docker container stop 1fa4ab2cf395
```

## 共享镜像

首先登陆docker：

```sh
$ docker login
```

之后将本地镜像与将要上传的镜像进行关联：

```sh
$ docker tag image username/repository:tag
```

例如：

```sh
docker tag friendlyhello gordon/get-started:part2
```

```sh
$ docker image ls

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
gordon/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```

### 发布镜像

```sh
docker push username/repository:tag
```

现在可以在任何机器上运行该镜像：

```sh
docker run -p 4000:80 username/repository:tag
```

如果该镜像不在本地，Docker会拉取该镜像：

```sh
$ docker run -p 4000:80 gordon/get-started:part2
Unable to find image 'gordon/get-started:part2' locally
part2: Pulling from gordon/get-started
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for gordon/get-started:part2
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

所有的依赖都在 `requirements.txt`中定义好了。

### 命令总结

```sh
docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyhello" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```



## 获取镜像

可以使用 `docker pull` 命令来从仓库获取所需要的镜像

```sh
sudo docker pull ubuntu:16.04
[sudo] figo 的密码： 
16.04: Pulling fro完成后，即可随时使用该镜像了，例如创建一个容器，让其中运行 bash 应用。

$ sudo docker run -t -i ubuntu:12.04 /bin/bash
root@fe7fc4bd8fc9:/#m library/ubuntu
9ff7e2e5f967: Pull complete 
59856638ac9f: Pull complete 
6f317d6d954b: Pull complete 
a9dde5e2a643: Pull complete 
Digest: sha256:cad5e101ab30bb7f7698b277dd49090f520fe063335643990ce8fbd15ff920ef
Status: Downloaded newer image for ubuntu:16.04

$ sudo docker attach ubuntu:12.04 #进入当前容器
```

完成后，即可随时使用该镜像了，例如创建一个容器，让其中运行 bash 应用。

```sh
$ sudo docker run -t -i ubuntu:12.04 /bin/bash
root@fe7fc4bd8fc9:/#
```

`-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开。

输入`exit`可以退出伪终端。

当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

## 守护态运行

更多的时候，需要让 Docker 容器在后台以守护态（Daemonized）形式运行。此时，可以通过添加 `-d` 参数来实现。

例如下面的命令会在后台运行容器。

```sh
$ sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
1e5535038e285177d5214659a068137486f96ee5c2e85a4ac52dc83f2ebe4147
```

容器启动后会返回一个唯一的 id，也可以通过 `docker ps` 命令来查看容器信息。

```sh
$ sudo docker ps
CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES
1e5535038e28  ubuntu:14.04  /bin/sh -c 'while tr  2 minutes ago  Up 1 minute        insane_babbage
```

要获取容器的输出信息，可以通过 `docker logs` 命令。

```sh
$ sudo docker logs insane_babbage
hello world
hello world
hello world
. . .
```

## 终止容器

可以使用 `docker stop` 来终止一个运行中的容器。

此外，当Docker容器中指定的应用终结时，容器也自动终止。 例如对于上一章节中只启动了一个终端的容器，用户通过 `exit` 命令或 `Ctrl+d` 来退出终端时，所创建的容器立刻终止。

终止状态的容器可以用 `docker ps -a` 命令看到。例如

```sh
sudo docker ps -a
CONTAINER ID        IMAGE                    COMMAND                CREATED             STATUS                          PORTS               NAMES
ba267838cc1b        ubuntu:14.04             "/bin/bash"            30 minutes ago      Exited (0) About a minute ago                       trusting_newton
98e5efa7d997        training/webapp:latest   "python app.py"        About an hour ago   Exited (0) 34 minutes ago                           backstabbing_pike
```

处于终止状态的容器，可以通过 `docker start` 命令来重新启动。

此外，`docker restart` 命令会将一个运行态的容器终止，然后再重新启动它。

## 进入容器

在使用 `-d` 参数时，容器启动后会进入后台。 某些时候需要进入容器进行操作，有很多种方法，包括使用`docker attach` 命令或 `nsenter` 工具等。

## 删除容器

可以使用 `docker rm` 来删除一个处于终止状态的容器。 例如

```
$sudo docker rm  trusting_newton
trusting_newton
```

如果要删除一个运行中的容器，可以添加 `-f` 参数。Docker 会发送 `SIGKILL` 信号给容器。

### 列出所有的容器 ID

```
docker ps -aq
```

### 停止所有的容器

```sh
docker stop $(docker ps -aq)
```

### 删除所有的容器

```sh
docker rm $(docker ps -aq)
```

### 删除所有的镜像

```sh
docker rmi $(docker images -q)
```

### 复制文件

```sh
docker cp mycontainer:/opt/file.txt /opt/local/docker cp /opt/local/file.txt mycontainer:/opt/
```



### 使用docker exec进入Docker容器

```sh
sudo docker ps  
sudo docker exec -it 775c7c9ee1e1 /bin/bash  

```

### 导出容器

如果要导出本地某个容器，可以使用 `docker export` 命令。

```sh
$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:14.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ sudo docker export 7691a814370e > ubuntu.tar
```

这样将导出容器快照到本地文件。

### 导入容器快照

可以使用 `docker import` 从容器快照文件中再导入为镜像，例如

```sh
$ cat ubuntu.tar | sudo docker import - test/ubuntu:v1.0
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
test/ubuntu         v1.0                9d37a6082e97        About a minute ago   171.3 MB
```

此外，也可以通过指定 URL 或者某个目录来导入，例如

```sh
$sudo docker import http://example.com/exampleimage.tgz example/imagerepo
```

> * 注：用户既可以使用 `docker load` 来导入镜像存储文件到本地镜像库，也可以使用 `docker import` 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。

### Docker下apt速度慢

```sh
sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list
apt-get clean
```

### Docker下Ubuntu安装不了Pip3

```sh
apt-get update
```

### 子镜像覆盖父镜像的ENTRYPOINT

父镜像Dockerfile

```sh
FROM ubuntu:14.04
ENTRYPOINT ["whoami"]
```

构建父镜像

```sh
sudo docker build -t kiwenlau/father .
```

子镜像Dockerfile

```sh
FROM kiwenlau/father
ENTRYPOINT ["hostname"]
```

构建子镜像:

```sh
sudo docker build -t kiwenlau/son .
```

运行父镜像:

```sh
sudo docker run kiwenlau/father
root
```

运行子镜像

```sh
sudo docker run kiwenlau/son
cb2b314c47db
```

可知， 父镜像输出了容器内的用户名， 而子镜像输出了容器的主机名。子镜像的ENTRYPOINT覆盖了父镜像的ENTRYPOINT

### Docker和主机共享文件夹

先在Mac本机创建一个共享文件夹
我是在`/Users/自己的用户名/`下创建了名为`withmydocker`的文件夹

```
docker run -it -v /Users/你自己的用户名/withmydocker:/home/withmymac --name <想要生成的container名> <image名> /bin/bash
```

