---
title: 如何用Git提交整个项目
date: 2019-11-23 19:56:31
index_img: /img/292DD11A-C1A1-473D-8628-2F16D39F4007.png
banner_img: /img/292DD11A-C1A1-473D-8628-2F16D39F4007.png
tags: [Git,版本控制]
categories: 代码
---

* 首先需要本地Git与自己的github账号使用SSH Key链接好，这部分的内容网上可以找到。

在github网页上创建好项目，本次项目名为ReBorn。

* 之后在本地打开终端，进入项目根目录,输入以下命令：

<!-- more -->

```sh
$ git init //初始化本地仓库  
$ git add *   //添加该目录下所有文件  
$ git commit -m "你的注释...."   //提交到本地仓库，并写一些注释  
$ git remote add test git@github.com:yourname/xxxx.git 
										或https://github.com/yourname/xxx.git
//连接远程仓库并建了一个名叫：test的别名，当然可以为其他名字。yourname为github名，xxx为项目名。
$ git push -u test master                              
//将本地仓库的文件提交到别名为test的地址的master分支下，-u为第一次提交，需要创建master分支，下次就不需要了 
```

本次我使用的命令就是：

```sh
$ git remote add test https://github.com/huxinpengfigo/ReBorn.git
```

常见问题：

> 1、在设置别名的时候，出现“fatal: remote origin already exists.”错误，说明该别名已经存在，可以另外建一个别名，或者使用“git remote rm origin”命令删除原来的别名，然后重新执行“git remote add origin git@github.com:yourname/xxxx.git”；
>
> 2、在提交的时候，出现“error: failed to push some refs to ‘git@github.com:xxx/xxx.git’ hint: Updates were rejected because the remote contains work that you do not have locally….”的错误，说明有冲突，远程仓库的版本比本地仓库的要信，所以要先进行更新，才能提交。使用“git pull git@github.com:xxx/xxx.git”命令进行更新，地址自己相应替换掉。
>
> 3、在提交时，“出现Updates were rejected because the tip of your current branch is behind...”的错误，可以直接强制提交（直接覆盖，对多人协作的项目很不友好，慎用）。
>
>   * push前先将远程repository修改pull下来
>
> ```sh
> $ git push -u test master -f
> ```
>
>   * push前先将远程repository修改pull下来
>
>
> ```sh
> $ git pull origin master
> 
> $ git push -u origin master
> ```
>
>   * 若不想merge远程和本地修改，可以先创建新的分支：
>
> ```sh
> $ git branch [name]
> ```
>
> 然后push
>
> ```sh
> $ git push -u origin [name]
> ```