---
title: golang flutter hover 构建好康的软件
date: 2023-01-08 19:32:26
updated: 2023-01-08 19:32:26
tags: [golang, flutter, hover, go-flutter]
categories: 
    - [golang]
keywords: 
description:
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
突然想弄个GUI的程序玩玩，但想着Gtk和Qt我这种菜鸡估计也整不明白，于是就盯上了flutter; 把UI当成web写这样能简单些，搭配着go-flutter+hover这两个包和golang我就能，前后端都简单的弄出来，还能把弄好的打包成appimage; 因此本文记录一下，从环境到打包出appimage的过程
# 安装
## golang 安装
我用着Archlinux 就直接用包管理器安装了， 毕竟滚动发行版包总是最新的
```bash
sudo pacman -S go
```
设置一下`go env`, 使得可以使用`go mod` 和 `go get` 国内提速
```bash
go env -w GO111MODULE="auto"
go env -w GOPROXY=https://goproxy.cn,direct
```
在`.bashrc`或`.zshrc`添加$GOPATH/bin到PATH, 具体加到哪里取决于用户shell是哪个
```bash
if [ -d "$HOME/go/bin" ];then
  export PATH="$PATH:$HOME/go/bin"
fi

```
添加完的话，go install 的包就可以之间用了

## flutter安装
下载压缩包 https://flutter.cn/docs/development/tools/sdk/releases?tab=linux
选 Beta channel (Linux)的 hover希望用beta而不是stable
```bash
# 我直接解压到了家目录
cd $HOME
# 俺写这篇文章时最新的
wget https://storage.flutter-io.cn/flutter_infra_release/releases/beta/linux/flutter_linux_3.7.0-1.1.pre-beta.tar.xz -c -q --show-progress
tar -xvf flutter_linux_3.7.0-1.1.pre-beta.tar.xz
```
写到.bashrc或.zshrc, 理由和go的一样
```bash
if [ -d "$HOME/flutter/bin" ];then
  export PATH="$PATH:$HOME/flutter/bin"
  export PUB_HOSTED_URL=https://pub.flutter-io.cn
  export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
fi
```
让.bashrc的更改生效
```bash
source $HOME/.bashrc
```
安装二进制开发文件
```bash
flutter precache
```
要是说flutter有新版就更新
```bash
flutter upgrade
```
## hover 安装
```bash
go get -u -a github.com/go-flutter-desktop/hover
go install github.com/go-flutter-desktop/hover
```
之后`hover`的二进制文件就到了`$HOME/go/bin`里面
# 食用
## 新建flutter项目
```bash
flutter create xxproject
#可以运行瞧瞧
# cd xxproject
# flutter run
```
## hover 初始化
```bash
cd xxproject
hover init xxproject
#运行
#hover run
```
## 编译打包appimage
配置flutter 允许编译 linux-desktop
```bash
flutter config --enable-linux-desktop
```
安装appimagetool,脚本
```bash
#!/bin/bash
wget "https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage"
 \
        -c -q --show-progress -O appimagetool
chmod a+x appimagetool
sudo cp appimagetool /usr/local/bin/appimagetool
rm appimagetool
```
安装好appimagetool后, 生成appimage打包信息
```bash
cd xxproject
hover init-packaging linux-appimage
```
编译生成linux-appimage
```bash
cd xxproject
hover build linux-appimage
```
### .packages 不存在 的错误
```bash
# 项目根目录生成.packages
cd xxproject
vim generate_dot_packages.sh
chmod +x generate_dot_packages.sh
./generate_dot_packages.sh
# 生成完之后重新编译打包
hover init-packaging linux-appimage
```

generate_dot_packages.sh 的内容
```bash
#!/bin/bash
cp .dart_tool/package_config.json .packages
```
# 执行生成的appimage
生成的appimage在项目目录的`go/build/outputs/linux-appimage-release`，这个目录里面
# 截图
![截图](https://cdn.basi-a.top/images/images_go-flutter-screenshot.webp)