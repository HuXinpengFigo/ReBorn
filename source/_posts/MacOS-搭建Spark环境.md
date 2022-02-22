---
title: MacOS 搭建Spark环境
date: 2021-09-16 14:37:50
index_img: /img/hadoop-vs-spark-main-1280x720.jpeg
banner_img: /img/hadoop-vs-spark-main-1280x720.jpeg
tags: [Spark,大数据]
---

### 下载与安装Spark

<!-- more -->

打开Spark的官网 [Downloads | Apache Spark](https://spark.apache.org/downloads.html)择合适自己版本的Spark安装包，下载完直接双击压缩包就会解压（建议安装一个解压软件），将其重命名为spark放到`/usr/local/Cellar`下面。

打开.bash_profile文件，添加：

```javascript
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bin
# Configure Spark to use Hadoop classpath
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
```

利用以下命令更新环境变量：

```sh
source  ~/.bash_profile
```

安装完毕，在终端输入：`spark-shell`，如果出现了下面的情况，代表安装成功了。

![Xnip2021-09-16_14-28-48](/img/Xnip2021-09-16_14-28-48.png)

