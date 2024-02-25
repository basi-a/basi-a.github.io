---
title: Arch 用debtap 安装 deb包
date: 2022-02-27 23:06:50
updated: 2022-12-15 17:59:50
tags: Linux
categories: 
    - [Linux,ArchLinux]
keywords: [ArchLinux,debtap,linux]
description: Arch 用debtap 安装 deb包
top_img:
comments:
cover:
toc:
toc_number:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
aside:
---
# 安装debtap

> paru -S debtap

## 更新debtap的数据库

> sudo debtap -u

_国内会非常的慢，不过由于 debtap 的 bin 文件是个 shell 脚本，所以改改脚本就行_

> sudo vim /bin/debtap

把脚本里面的 debian 源和 ubuntu源换成 国内镜像源就行了

如：把脚本里 `http://ftp.debian.org/` 和 `http://archive.ubuntu.com/` 都换成 `https://mirrors.ustc.edu.cn/`

## 使用debtap转换deb包

> debtap xxx.deb

按照提示输入包名，license。包名自己起，license 填GPL之类的，完成后会在 deb 包相同目录下生成 pacman 可安装的包， 如`xxx.pkg.tar.zst`

# 用pacman安装

> sudo pacman -U xxx.tar.zst

安装好的包，和自己起的包名是一样的

