---
layout: post
title:  "如何不通过iTunes将Mac上的音乐同步到iPad"
date:   2022-05-24 20:09:10 +0800
categories: ios macos
---

本文记录了如何不通过iTunes（或Finder，如果是新系统的话）将苹果电脑上的文件（音乐、视频等）同步到iPad。本文以同步音乐为例。

1. 在Terminal中`cd`到音乐文件夹（如`~/Music`），使用[fd](https://github.com/sharkdp/fd)命令列出所有音乐并传给zip打包，假设打包为`share.zip`：`fd -emp4 -etma -em4a -emp3 -d1 . | zip -0qT share.zip -@`
2. 在Terminal中输入`ifconfig | grep 192 | awk '{ print $2 }'`确认自己在局域网中的IP地址。
3. 使用`python3 -m http.server 9000`建立一个http服务器，这里建立在9000端口上。
4. 将iPad连接至与Mac同一局域网。打开iPad的Safari浏览器，在地址栏输入`http://192.168.0.xxx:9000`，其中`192.168.0.xxx`表示在第2步中确认的IP地址。
5. 在`Directory listing for /`下面找到`share.zip`，单击下载。下载完毕后应该在`Files`应用中的`On My iPad/Downloads`下面找到。
6. 单击`share.zip`，此时会自动解压为`share`文件夹。单击`share`文件夹进入，右上角点击`Select`，然后左上角点击`Select All`全选，然后下面点击`Move`，选择位置，例如移动到`On My iPad/Music`，音乐就都移动过去了。
7. 删除`share`文件夹和`share.zip`。

虽然步骤有点多，熟练了也不是很麻烦。第3步中建立的服务器可以常开着，以便随时同步。
