---
title: linux 脚本安装 Apifox
date: 2022-12-29 16:41:27
updated: 2022-12-29 16:41:27
tags: 
    - [Linux, shell]
categories: 
    - [Linux]
    - [shell]
keywords: [Apifox, shell脚本安装]
description: Archlinux 上使用 shell 脚本安装 Apifox
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
# 前言
为了方便自己使用apifox来调试api, 从而写了个shell脚本来在linux上安装appimage版的apifox
下载好的appimage包，重命名放到`/usr/local/bin`下，方便自己通过dmenu打开
# 脚本
```bash
#!/bin/bash
URL=https://cdn.apifox.cn/download/Apifox-linux-latest.zip
ZIPNAME=Apifox-linux-latest.zip
DIR=Apifox-linux-latest
PKGNAME=Apifox.AppImage
install(){
        wget -q ${URL} --show-progress
        unzip ${ZIPNAME} -d ${DIR}
        chmod +x ${DIR}/${PKGNAME}
        sudo cp ${DIR}/${PKGNAME} /usr/local/bin/apifox
        rm -rf ${DIR} ${ZIPNAME}
}
main(){
        install
}
main
```
