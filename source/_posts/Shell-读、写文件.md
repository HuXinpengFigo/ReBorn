---
title: Shell 读、写文件
date: 2021-04-24 16:19:13
index_img: /img/shell.png
banner_img: /img/shell.png
tags: [Linux,Shell]
categories: 代码
---

### 读文件

读文件主要使用`cat`命令来完成。

<!-- more -->

首先我们要读的对象为`names.txt`，内容为：

```
hello_world1
hello_world2
hello_world3
hello_world4
hello_world5
```

使用`cat`命令的值（即为文件内容）作为返回值。

```sh
#!/bin/zsh

names=$(cat ./names.txt)

echo $names
```

> 此处names赋值时`=`前后不能加括号，`#!/bin/zsh`表示使用的是MacOS的`zsh`Shell，在Linux可以改为`#!/bin/bash`。

我们可以看到每一行的结果以空格分隔的形式打出，这意味着我们可以使用一个for循环去遍历他们：

```sh
#!/bin/zsh

names=$(cat ./names.txt)

for line in $names
do
  echo $line
done
```

### 写文件

写文件使用`>>`或者`>`来完成，其中`>>`表示**追加**内容，`>`表示**覆盖**原有内容。

我们要写的对象为`target.txt`:

```sh
#!/bin/zsh

names=$(cat ./names.txt)

for line in $names
do
  echo $line >> ./target.txt
done
```

