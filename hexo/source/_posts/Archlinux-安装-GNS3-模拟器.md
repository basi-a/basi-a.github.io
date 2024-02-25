---
title: ArchLinux 本地安装 GNS3 模拟器
date: 2023-06-02 18:25:24
updated: 2023-06-02 18:25:24
tags: archlinux, gns3
categories: 
- [Linux,ArchLinux]
- [GNS3]
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
# GNS3是什么
[GNS3](https://gns3.com/) 是思科公司开源的网络模拟器，与`packet tracer`不同，`packet tracer`是纯软件模拟，而`GNS3`是更高级的模拟器，网络设备都是跑的虚拟机，需要自行准备网络设备的镜像；`windows`和`macos`的可以运行在虚拟机里面，此时网络设备就是嵌套虚拟化的；而`Linux`直接运行于本地，此时网络设备可以是`qemu`的虚拟机，`docker`容器，或者`Vbox`的虚拟机；总的来说，这是一个很强大的网络模拟器。
# 安装GNS3
因为我只有这一台装着`ArchLinux`的笔记本，所以其他系统，以及其他Linux发行版的我就不写了。
## 从AUR安装需要的包
**要提前安装好libvirt 能用KVM/QEMU**
```bash
paru -S gns3-gui gns3-server dynamips dnsmasq ubridge vpvs wireshark
```
## 启动服务
这里用的用户，就是要运行GNS3的用户，用不着sudo提权
```bash
systemctl enable gns3-server@USER --now
```
此用户需要属于`libvirt`组，免得开个KVM虚拟机还要提权； 还要属于`docker`组，同理
```bash
sudo usermod -a -G libvirt USER
sudo usermod -a -G docker USER
```
## 设置要用的终端
用于调试设备，默认是`xterm`，设置里面可以改，我设置成了`KDE`自带的`konsole`
## GNS3使用wireshark抓包
`GNS3`是可以和`wireshark`一起用的，用来抓取拓扑设备间的数据包。
```bash
mkdir $HOME/GNS3/wireshark
ln -s /usr/bin/wireshark $HOME/GNS3/wireshark
```

`wireshark`要用的话，此用户还得属于`wireshark`组
```bash
sudo usermod -a -G wireshark USER
```
# 导入镜像
_需要自己下载_
这里找到两个提供资源的

https://ccie.lol/blog/2016/07/03/cisco-ios-image-download/

https://bbs.hh010.com/forum-ios-1.html

下载之后，到GNS3里面添加就好
# 初体验
填加了一个设备，开了起来，让我逝着玩一玩
![gns3](https://cdn.basi-a.top/images/images_gns3.webp)
