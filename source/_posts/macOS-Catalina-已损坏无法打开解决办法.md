---
title: macOS Catalina 已损坏无法打开解决办法
date: 2019-11-26 17:46:50
index_img: /img/kissclipart-logo-de-mac-os-clipart-macos-computer-icons-81d21f0ecb21adb8.png
banner_img: /img/m8c20gchf7231.jpg
tags: [MacOS]
categories: MacOS
---

App 在macOS Catalina下提示已损坏无法打开解决办法：

> 首先打开“任意来源安装”

<!-- more -->

1. 打开终端；
2. 输入以下命令，回车；
   `sudo spctl --master-disable`

3. 在设置里选择“从任意来源安装”

![](/img/Xnip2019-11-26_17-49-20.png)

>  还是无效的话：

1. 打开终端；
2. 输入以下命令，回车；
   `sudo xattr -d com.apple.quarantine /Applications/xxxx.app`
   注意：`/Applications/xxxx.app` 换成你的App路径
3. 重启App即可。

