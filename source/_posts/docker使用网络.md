---
title: docker使用网络
date: 2020-03-31 16:43:48
index_img: /img/dockera.png
banner_img: /img/docker-cloud-twitter-card.png
tags: [docker]
categories: 代码
---



## 外部访问容器

容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 `-P` 或 `-p` 参数来指定端口映射。

<!-- more -->

当使用 -P 标记时，Docker 会随机映射一个 `49000~49900` 的端口到内部容器开放的网络端口。

使用 `docker ps` 可以看到，本地主机的 49155 被映射到了容器的 5000 端口。此时访问本机的 49155 端口即可访问容器内 web 应用提供的界面。

```sh
$ sudo docker run -d -P training/webapp python app.py
$ sudo docker ps -l
CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS                    NAMES
bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up 2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse
```

同样的，可以通过 `docker logs` 命令来查看应用的信息。

```sh
$ sudo docker logs -f nostalgic_morse
* Running on http://0.0.0.0:5000/
10.0.2.2 - - [23/May/2014 20:16:31] "GET / HTTP/1.1" 200 -
10.0.2.2 - - [23/May/2014 20:16:31] "GET /favicon.ico HTTP/1.1" 404 -
```

-p（小写的）则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有`ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort`。

### 1.1 映射所有接口地址

使用 `hostPort:containerPort` 格式本地的 5000 端口映射到容器的 5000 端口，可以执行

```sh
$ sudo docker run -d -p 5000:5000 training/webapp python app.py
```

此时默认会绑定本地所有接口上的所有地址。

### 1.2 映射到指定地址的指定端口

可以使用 `ip:hostPort:containerPort` 格式指定映射使用一个特定地址，比如 localhost 地址 127.0.0.1

```sh
$ sudo docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

### 1.3 映射到指定地址的任意端口

使用 `ip::containerPort` 绑定 localhost 的任意端口到容器的 5000 端口，本地主机会自动分配一个端口。

```sh
$ sudo docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

还可以使用 udp 标记来指定 udp 端口

```sh
$ sudo docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```

### 1.4 查看映射端口配置

使用 `docker port` 来查看当前映射的端口配置，也可以查看到绑定的地址

```sh
$ docker port nostalgic_morse 5000
127.0.0.1:49155.
```

注意：

- 容器有自己的内部网络和 ip 地址（使用 `docker inspect` 可以获取所有的变量，Docker 还可以有一个可变的网络配置。）
- -p 标记可以多次使用来绑定多个端口

例如

```sh
$ sudo docker run -d -p 5000:5000  -p 3000:80 training/webapp python app.py
```

## 2、容器互联

容器的连接（linking）系统是除了端口映射外，另一种跟容器中应用交互的方式。

该系统会在源和接收容器之间创建一个隧道，接收容器可以看到源容器指定的信息。

### 2.1 自定义容器命名

连接系统依据容器的名称来执行。因此，首先需要自定义一个好记的容器命名。

虽然当创建容器的时候，系统默认会分配一个名字。自定义命名容器有2个好处：

- 自定义的命名，比较好记，比如一个web应用容器我们可以给它起名叫web
- 当要连接其他容器时候，可以作为一个有用的参考点，比如连接web容器到db容器

使用 `--name` 标记可以为容器自定义命名。

```sh
$ sudo docker run -d -P --name web training/webapp python app.py
```

使用 `docker ps` 来验证设定的命名。

```sh
$ sudo docker ps -l
CONTAINER ID  IMAGE                  COMMAND        CREATED       STATUS       PORTS                    NAMES
aed84ee21bde  training/webapp:latest python app.py  12 hours ago  Up 2 seconds 0.0.0.0:49154->5000/tcp  web
```

也可以使用 `docker inspect` 来查看容器的名字

```sh
$ sudo docker inspect -f "{{ .Name }}" aed84ee21bde
/web
```

注意：容器的名称是唯一的。如果已经命名了一个叫 web 的容器，当你要再次使用 web 这个名称的时候，需要先用`docker rm` 来删除之前创建的同名容器。

在执行 `docker run` 的时候如果添加 `--rm` 标记，则容器在终止后会立刻删除。注意，`--rm` 和 `-d` 参数不能同时使用。

### 2.2 容器互联

使用 `--link` 参数可以让容器之间安全的进行交互。

下面先创建一个新的数据库容器。

```sh
$ sudo docker run -d --name db training/postgres
```

删除之前创建的 web 容器

```sh
$ docker rm -f web
```

然后创建一个新的 web 容器，并将它连接到 db 容器

```sh
$ sudo docker run -d -P --name web --link db:db training/webapp python app.py
```

此时，db 容器和 web 容器建立互联关系。

`--link` 参数的格式为 `--link name:alias`，其中 `name` 是要链接的容器的名称，`alias` 是这个连接的别名。

使用 `docker ps` 来查看容器的连接

```sh
$ docker ps
CONTAINER ID  IMAGE                     COMMAND               CREATED             STATUS             PORTS                    NAMES
349169744e49  training/postgres:latest  su postgres -c '/usr  About a minute ago  Up About a minute  5432/tcp                 db, web/db
aed84ee21bde  training/webapp:latest    python app.py         16 hours ago        Up 2 minutes       0.0.0.0:49154->5000/tcp  web
```

可以看到自定义命名的容器，db 和 web，db 容器的 names 列有 db 也有 web/db。这表示 web 容器链接到 db 容器，web 容器将被允许访问 db 容器的信息。

Docker 在两个互联的容器之间创建了一个安全隧道，而且不用映射它们的端口到宿主主机上。在启动 db 容器的时候并没有使用 `-p` 和 `-P` 标记，从而避免了暴露数据库端口到外部网络上。

Docker 通过 2 种方式为容器公开连接信息：

- 环境变量
- 更新 `/etc/hosts` 文件

使用 `env` 命令来查看 web 容器的环境变量

```sh
$ sudo docker run --rm --name web2 --link db:db training/webapp env
. . .
DB_NAME=/web2/db
DB_PORT=tcp://172.17.0.5:5432
DB_PORT_5000_TCP=tcp://172.17.0.5:5432
DB_PORT_5000_TCP_PROTO=tcp
DB_PORT_5000_TCP_PORT=5432
DB_PORT_5000_TCP_ADDR=172.17.0.5
. . .
```

其中 DB_ 开头的环境变量是供 web 容器连接 db 容器使用，前缀采用大写的连接别名。

除了环境变量，Docker 还添加 host 信息到父容器的 `/etc/hosts` 的文件。下面是父容器 web 的 hosts 文件

```sh
$ sudo docker run -t -i --rm --link db:db training/webapp /bin/bash
root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7  aed84ee21bde
. . .
172.17.0.5  db
```

这里有 2 个 hosts，第一个是 web 容器，web 容器用 id 作为他的主机名，第二个是 db 容器的 ip 和主机名。 可以在 web 容器中安装 ping 命令来测试跟db容器的连通。

```sh
root@aed84ee21bde:/opt/webapp# apt-get install -yqq inetutils-ping
root@aed84ee21bde:/opt/webapp# ping db
PING db (172.17.0.5): 48 data bytes
56 bytes from 172.17.0.5: icmp_seq=0 ttl=64 time=0.267 ms
56 bytes from 172.17.0.5: icmp_seq=1 ttl=64 time=0.250 ms
56 bytes from 172.17.0.5: icmp_seq=2 ttl=64 time=0.256 ms
```

用 ping 来测试db容器，它会解析成 `172.17.0.5`。 *注意：官方的 ubuntu 镜像默认没有安装 ping，需要自行安装。

用户可以链接多个父容器到子容器，比如可以链接多个 web 到 db 容器上。

# 高级网络配置

## 1、快速配置指南

下面是一个跟 Docker 网络相关的命令列表。

其中有些命令选项只有在 Docker 服务启动的时候才能配置，而且不能马上生效。

- -b BRIDGE or –bridge=BRIDGE –指定容器挂载的网桥
- –bip=CIDR –定制 docker0 的掩码
- -H SOCKET… or –host=SOCKET… –Docker 服务端接收命令的通道
- –icc=true|false –是否支持容器之间进行通信
- –ip-forward=true|false –请看下文容器之间的通信
- –iptables=true|false –禁止 Docker 添加 iptables 规则
- –mtu=BYTES –容器网络中的 MTU

下面2个命令选项既可以在启动服务时指定，也可以 Docker 容器启动（docker run）时候指定。在 Docker 服务启动的时候指定则会成为默认值，后面执行 docker run 时可以覆盖设置的默认值。

- –dns=IP_ADDRESS… –使用指定的DNS服务器
- –dns-search=DOMAIN… –指定DNS搜索域

最后这些选项只有在 docker run 执行时使用，因为它是针对容器的特性内容。

- -h HOSTNAME or –hostname=HOSTNAME –配置容器主机名
- –link=CONTAINER_NAME:ALIAS –添加到另一个容器的连接
- –net=bridge|none|container:NAME_or_ID|host –配置容器的桥接模式
- -p SPEC or –publish=SPEC –映射容器端口到宿主主机
- -P or –publish-all=true|false –映射容器所有端口到宿主主机

## 2、配置 DNS

Docker 没有为每个容器专门定制镜像，那么怎么自定义配置容器的主机名和 DNS 配置呢？ 秘诀就是它利用虚拟文件来挂载到来容器的 3 个相关配置文件。

在容器中使用 mount 命令可以看到挂载信息：

```sh
$ mount
...
/dev/disk/by-uuid/1fec...ebdf on /etc/hostname type ext4 ...
/dev/disk/by-uuid/1fec...ebdf on /etc/hosts type ext4 ...
tmpfs on /etc/resolv.conf type tmpfs ...
...
```

这种机制可以让宿主主机 DNS 信息发生更新后，所有 Docker 容器的 dns 配置通过 `/etc/resolv.conf` 文件立刻得到更新。

如果用户想要手动指定容器的配置，可以利用下面的选项。

`-h HOSTNAME or --hostname=HOSTNAME` 设定容器的主机名，它会被写到容器内的 `/etc/hostname` 和`/etc/hosts`。但它在容器外部看不到，既不会在 `docker ps` 中显示，也不会在其他的容器的 `/etc/hosts` 看到。

`--link=CONTAINER_NAME:ALIAS` 选项会在创建容器的时候，添加一个其他容器的主机名到 `/etc/hosts` 文件中，让新容器的进程可以使用主机名 ALIAS 就可以连接它。

`--dns=IP_ADDRESS` 添加 DNS 服务器到容器的 `/etc/resolv.conf` 中，让容器用这个服务器来解析所有不在`/etc/hosts` 中的主机名。

`--dns-search=DOMAIN` 设定容器的搜索域，当设定搜索域为 `.example.com` 时，在搜索一个名为 host 的主机时，DNS 不仅搜索host，还会搜索 `host.example.com`。 注意：如果没有上述最后 2 个选项，Docker 会默认用主机上的 `/etc/resolv.conf` 来配置容器。

