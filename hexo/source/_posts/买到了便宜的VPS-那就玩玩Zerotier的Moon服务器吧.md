---
title: 买到了便宜的VPS, 那就玩玩Zerotier的Moon服务器吧
date: 2024-02-22 22:04:48
updated: 2024-02-22 22:04:48
tags: Zerotier
categories:
keywords: Moon
description: 买到了便宜的VPS, 那就玩玩Zerotier的Moon服务器吧
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

> 今天买到了便宜的vps，虽然`2 vCPU 2 GiB 40 GiB RAID-10 HS SSD`, 看似不强，但是够我拿来做`Zerotier`的卫星服务器了，另外还可以拿来开`frps`



> Zerotier 三个等级：
>
> PLANET 行星服务器，Zerotier 各地的根服务器，有日本、新加坡等地
> MOON 卫星级服务器，用户自建的私有根服务器，起到中转加速的作用
> LEAF 相当于各个枝叶，就是每台连接到该网络的机器节点。

# 全节点操作

以下操作只针对Linux, windows的有图形配置，很简单;

~~至于mac, 别问问就是，穷鬼买不起；~~

## 安装zerotier-one

```bash
curl -s https://install.zerotier.com/ | sudo bash
# for Archlinux
# paru -S zerotier-one
```

## 启动并加入网络

```bash
sudo systemctl enable zerotier-one --now
sudo zerotier-cli join ${Network-ID}
```

之后管理面板找到`Members`对这些节点授权

# Moon节点

这个必须是个公网的服务器, 以下操作就都用root用户吧

## 生成节点配置

```bash
sudo -i
cd /var/lib/zerotier-one/
zerotier-idtool initmoon /var/lib/zerotier-one/identity.public > moon.json
```

## 编辑配置

```bash
vim moon.json
```

修改这个{`"roots": [{"stableEndpoints": ["ip/port"]}]}`，方括号里面写`"公网IP/9993"`, 配置文件结构就像我下面这张图

![moon-set-stableEndpoints](https://cdn.basi-a.top/images/moon-set-stableEndpoints.png)

## 生成签名

```bash
zerotier-idtool genmoon moon.json
```

下面是我的输出

> wrote 000000844acc14c1.moon (signed world with timestamp 1708605206488)

创建`moons.d`目录，把上面生成的挪进去

```bash
mkdir moons.d
mv 000000844acc14c1.moon moons.d
```

## 重启zerotier

```bash
systemctl restart zerotier-one
```

# 非Moon节点

## 操作方法A

执行下面的脚本就可以获取`moon`信息

```bash
#!/bin/bash
IP=$1
ztaddr=$(sudo zerotier-cli listpeers | grep $IP | awk '{print $3}')
sudo zerotier-cli orbit $ztaddr $ztaddr
```

其实重点就是`zerotier-cli listpeers`找到这玩意输出中作为Moon服务器的那行的IP，所对应的`<ztaddr>`列的值

然后`zerotier-cli orbit` 后面俩参数都是这个`<ztaddr>`列的值，这个过程很简单，windows我就不写脚本了，直接`powershell`（管理员模式哦）手操一下得了

上面操作，成功的话会输出这个

> 200 orbit OK

然后再看一眼`listpeers`，就会发现作为moon服务器的IP,对应的`<role>`就是`MOON`, `listmoons`会输出`moon`服务器的信息

```bash
sudo zerotier-cli listpeers
sudo zerotier-cli listmoons
```

## 操作方法B

拿着Moon节点生成的`/var/lib/zerotier-one/moons.d/000000844acc14c1.moon`，扔到非Moon节点的`/var/lib/zerotier-one/moons.d/`, 然后重启服务

当然我觉得这个方法不咋优雅
