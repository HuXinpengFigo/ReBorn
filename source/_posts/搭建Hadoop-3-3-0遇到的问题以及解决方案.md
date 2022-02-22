---
title: 搭建Hadoop 3.3.0遇到的问题以及解决方案
date: 2021-10-13 15:00:28
index_img: /img/hadoop-vs-spark-main-1280x720.jpeg
banner_img: /img/hadoop-vs-spark-main-1280x720.jpeg
tags: [Hadoop,大数据]
---

### 启动`start-dfs.sh`出现的报错

<!-- more -->

- 出现：

```sh
ERROR: Attempting to operate on hdfs namenode as root
ERROR: but there is no HDFS NAMENODE USER defined. Aborting operation.
Starting datanodes
```

#### 解决方法

在文件**开头空白**处添加以下内容：

`start-dfs.sh, stop-dfs.sh`

```xml
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root 
```

`start-yarn.sh, stop-yarn.sh`

```xml
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

> 集群中所有机子都需要修改，之后不在赘述。

- 出现

```sh
Starting namenodes on | hadoop100]
hadoop100: Warning: Permanently added'hadoop100,192.168.79.5' (ECDSA) to the list of known hosts.
hadoop100:Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
localhost: Warning: Permanently added 'localhost'( ECDSA)to the list of known hosts.
localhost:Permission denied (publickey, gssapi-keyex, gssapi-with-mic, password)
```

此处是由于`shh`并没有权利访问`localhost`，也就是本地key无法访问本地，需要将本地key加入`authorized_kyes`里面。

```sh
cd ~/.ssh/
cat id_rsa.pub >> authorized_keys
```

#### Hadoop 2.7 与 3.0 特性区别

在Hadoop 3.0以上，不存在`slaves`文件，使用`worker`文件替代，如果出现在master机上启动dfs但是worker机上不显示DataNode则说明`worker`文件未修改好。

#### Hadoop 3.0默认端口区别

- `hdfs`的web页面默认端口是`9870 `

- `yarn`的web页面端口是`8088`
