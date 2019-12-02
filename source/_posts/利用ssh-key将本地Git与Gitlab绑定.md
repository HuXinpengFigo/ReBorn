---
title: 利用ssh key将本地Git与Gitlab绑定
date: 2019-11-25 12:21:30
index_img: /img/Xnip2019-11-25_14-02-30.png
banner_img: /img/Xnip2019-11-25_12-27-46.png
tags: [Git,版本控制]
categories: 代码
---

# 利用ssh key将本地Git与Gitlab绑定

### 生成ssh key

> 对于Mac系统，ssh是默认安装的，我们只需要生成ssh key并查看就好了

<!-- more -->

生成ssh key：

```sh
ssh-keygen -t rsa
```

> 表示我们指定 RSA 算法生成密钥，然后敲三次回车键，期间不需要输入密码，之后就就会生成两个文件，分别为`id_rsa`和`id_rsa.pub`，即密钥`id_rsa`和公钥`id_rsa.pub`。

复制ssh key公钥：

```sh
pbcopy < ~/.ssh/id_rsa.pub
```

### 添加ssh key到Gitlab

* 进入Gitlab主页，右上角选择Settings进入

<img src="/img/Xnip2019-11-24_19-49-47.png" style="zoom: 50%;" />

* 左边选择SHH Keys

![](/img/Xnip2019-11-24_19-50-06.png)

* 在右边框中填入之前复制的ssh公钥

![](/img/Xnip2019-11-24_19-50-36.png)



### Gitlab基础操作

 ##### Git global setup

```sh
git config --global user.name "胡鑫鹏"
git config --global user.email "figohxp@163.com"
```

 ##### Create a new repository

```sh
git clone git@git.lug.ustc.edu.cn:NisemonoFigo/nisemonofigo.gitlab.io.git
cd nisemonofigo.gitlab.io
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

 ##### Push an existing folder

```sh
cd existing_folder
git init
git remote add origin git@git.lug.ustc.edu.cn:NisemonoFigo/nisemonofigo.gitlab.io.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

 ##### Push an existing Git repository

```sh
cd existing_repo
git remote rename origin old-origin
git remote add origin git@git.lug.ustc.edu.cn:NisemonoFigo/nisemonofigo.gitlab.io.git
git push -u origin --all
git push -u origin --tags
```

