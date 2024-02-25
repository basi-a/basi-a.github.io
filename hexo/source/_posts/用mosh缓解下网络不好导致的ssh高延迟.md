---
title: 用mosh缓解下网络不好导致的ssh高延迟
date: 2023-09-30 12:51:02
updated: 2023-09-30 12:51:02
tags: zerotier, mosh, ssh
categories: Linux
keywords: mosh
description: 用mosh缓解下网络不好导致的ssh高延迟
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
> 前几天买了个`香橙派3B`, 想着搭配着`zerotier`, 我就有能随时随地用的Linux环境了, 在家里还好, 别说局域网直连，就算流量从`zerotier`走一圈,也一点不卡
> 但是呢, 只要到了网络比较复杂的地方，那个延迟啊，啧啧啧
> 于是呢经过一番冲浪, 发现了`mosh`, 算是解决了我被`ssh`卡的头皮发麻的问题
# mosh
[mosh](https://mosh.org/)
# 安装方法
```bash
sudo apt install mosh
```
> 其他Linux发行版也是如此，包管理里面基本上都有

## Windows客户端获取
[mosh-client and mosh download](https://github.com/felixse/FluentTerminal/tree/master/Dependencies/MoshExecutables/x64)
mosh 官方是没有提供Windows的客户端以及服务端的，但是`felixse/FluentTerminal`这个终端，有编译好的客户端可执行文件，下载下来，添加环境变量就能用
# 连接方法
## 方法1
```bash
mosh basi@IP
```
> 就和命令行使用ssh连接一样，mosh会先用ssh登录，之后开启mosh-server，之后就是UDP协议的交互了
> 还可以加上`-p PORT`来指定使用哪个udp端口, 为了能用呢防火墙要放行这个端口，默认不写的话是udp 60000-61000,记得防火墙要放行
```bash
mosh basi@IP -p PORT
```
## 方法2
> 先ssh连接进去，然后手动启动mosh-server,拿到MOSH_KEY再连接
```bash
ssh basi@IP
mosh-server -new -p PORT
```
```bash
export MOSH_KEY="KEY" && mosh-client IP PORT
```
```bash
# windows 命令行连接
$env:MOSH_KEY="KEY"
mosh-client IP PORT
```