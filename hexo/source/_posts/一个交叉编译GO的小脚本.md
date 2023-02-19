---
title: 一个交叉编译GO的小脚本
date: 2022-08-06 15:44:05
updated: 2022-12-15 18:14:05
tags: [go,bash]
categories: 
    - [Linux]
    - [shell]
keywords: 交叉编译
description: 一个交叉编译GO的小脚本
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
# 所谓交叉编译
> 所谓交叉编译，就是在一个平台上去编译出其他操作系统和CPU架构的二进制文件供其他环境使用。比如linux下写出的玩意想编译完能在win上用，这就是交叉编译。

> 最近在看GO的基础，突然想把在linux上用GO写出来的玩意，弄到win上用，于是有了这么个小脚本，虽然没啥大用(╯‵□′)╯︵┻━┻
# 垃圾脚本
```bash
#!/usr/bin/bash
selectOS(){
    echo "Please select the target OS"  
    echo "1. windows, 2. linux, 3. freebsd, 4. darwin"
    read -p "Target OS > " os
    case $os in
        1) echo "selected windows"
        target_os=windows
        ;;
        2) echo "selected linux"
        target_os=linux
        ;;
        3) echo "selected freebsd"
        target_os=freebsd
        ;;
         4) echo "selected darwin"
        target_os=darwin
        ;;
        *) echo "A wrong input !!!"
        exit
        ;;
    esac
}

selectArch(){
    echo "Please select the target CPU architecture"  
    echo "1. amd64, 2. arm, 3. 386"
    read -p "Target CPU ARCH > " arch
    case $arch in
        1) echo "selected amd64"
        target_arch=amd64
        ;;
        2) echo "selected arm"
        target_arch=arm
        ;;
        3) echo "selected 386"
        target_arch=386
        ;;
        *) echo "A wrong input !!!"
        exit
        ;;
    esac
}

inputOutputname(){
    echo "Please input the output name "
    read -p "Outputname > " outputname
}

startBuild(){
    enable=0
    CGO_ENABLED=$enable GOOS=$target_os GOARCH=$target_arch go build -o $outputname $1
}

run(){
    selectOS
    selectArch
    inputOutputname
    startBuild
}
run
```
效果如下：
![效果](https://raw.githubusercontent.com/basi-a/GoCrossCompilation/main/HowToUse.png)
github 仓库 https://github.com/basi-a/GoCrossCompilation
gitee 仓库 https://gitee.com/basi-a/GoCrossCompilation