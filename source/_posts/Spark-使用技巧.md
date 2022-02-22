---
title: Spark Hive 使用技巧
date: 2021-10-17 11:38:49
index_img: /img/Apache_Spark_logo.svg.png
banner_img: /img/hadoop-vs-spark-main-1280x720.jpeg
tags: [Spark,大数据,conda]
---

## 解决Spark运行期间LOG输出INFO过多

运行`spark-sql`或者`spark-shell`的时候经常会出现满屏的`INFO`输出，影响对结果的阅读。

![Xnip2021-10-17_11-42-18](/img/Xnip2021-10-17_11-42-18.png)

这个时候只需要在`spark-conf/conf/`下设置好`log4j.properties`文件中的`log4j.rootCategory`就可以解决，甚至进行深度研究后可以自由设置应该展示哪个部分的信息。

### 复制`log4j.properties.template`为`log4j.properties`

```sh
cp log4j.properties.template log4j.properties
vim log4j.properties
```

原文件设置：

```properties
log4j.rootCategory=INFO, console
```

改为：

```properties
log4j.rootCategory=WARN, console
```

效果：

![Xnip2021-10-17_11-46-40](/img/Xnip2021-10-17_11-46-40.png)

## Spark/Hive运行`.sql`文件

### Spark运行`.sql`

`spark`运行`.sql`文件的组件为 `spark-sql`，使用方法：

```sh
spark-sql -f example.sql
```

### Hive运行`.sql`

`hive`运行`.sql`文件的方法有两种。

#### Hive直接执行

```sh
hive -f example.sql
```

### `hive shell`执行

进入`hive shell`之后使用`source`命令执行

```sh
hive

>>进入hive shell了

source example.sql
```

