---
title: Hive启动Metastore的3种方式
date: 2021-10-15 15:57:11
index_img: /img/0a6e3ff0d40e42bc581c88b6cada856825ea325br1-512-512v2_uhq.jpeg
banner_img: /img/Getting-started-with-apache-hive.jpeg
tags: [Hive,大数据]
---

### 启动`metastore`

```sh
hive --service metastore -p <your port>
```

<!-- more -->

后台启动`metastore`

```sh
bin/hive --service metastore 2>&1 >> /var/log.log &
```

> 此方法启动的`metastore`在退出`ssh`连接之后还是会被关闭

后台启动`metastore`并且关闭`ssh`连接后依然存在

```sh
nohup bin/hive --service metastore 2>&1 >> /var/log.log &
```
