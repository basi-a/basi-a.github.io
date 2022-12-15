---
title: Arch的双显卡驱动
date: 2022-02-09 17:55:36
updated: 2022-12-15 17:55:36
tags: [arch,linux]
categories: 
    - [linux,Archinux]
keywords: [archlinux,双显卡]
description: Arch的双显卡驱动
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
# Arch 的双显卡驱动 (xorg)
**注意 ：本文两种方案不可共存！！**
# optimus manager 方案
## I卡驱动

不用装intel的驱动，直接用mesa的就行

## N卡驱动

编辑 `pacman.conf` 开启 32位软件源（multilib）
```bash
sudo vim /etc/pacman.conf
```
删掉这两行的注释
```text
[multilib]
Include = /etc/pacman.d/mirrorlist
```
##同步软件包数据库

```bash
sudo pacman -Syy
```
##安装Nvidia显卡闭源驱动 (非自定义内核)
```bash
sudo pacman -S nvidia nvidia-prime nvidia-settings nvidia-utils opencl-nvidia lib32-nvidia-utils lib32-opencl-nvidia
```


## 双显卡驱动切换工具
使用的是 `optimus-manager` + `bbswitch`

安装optimus-manager 和 bbswitch
```bash
sudo pacman -S optimus-manager bbswitch
```
### 图形化切换工具
```bash
paru -S optimus-manager-qt
```
不用这个可以复制github上optimus-manager作者给的的配置，自己填写

***********
当使用dwm 且直接用startx时，需要在`～/.xinitrc` 中加上 
```text
/usr/bin/prime-offload &
```
另外还要保证logout时，`/usr/bin/prime-switch` 以root执行
********
```bash
optimus-manager --switch Nvidia
optimus-manager --switch integrated
optimus-manager --print mode
```
以上分别是切换N卡，I卡，以及查看当前显卡模式
*********************
# PRIME 方案
## 双卡驱动同 `optimus` 方案
## 添加N卡配置文件
```bash
sudo vim /etc/udev/rules.d/80-nvidia-pm.rules
```
```text
# Enable runtime PM for NVIDIA VGA/3D controller devices on driver bind
ACTION=="bind", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x030000", TEST=="power/control", ATTR{power/control}="auto"
ACTION=="bind", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x030200", TEST=="power/control", ATTR{power/control}="auto"

# Disable runtime PM for NVIDIA VGA/3D controller devices on driver unbind
ACTION=="unbind", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x030000", TEST=="power/control", ATTR{power/control}="on"
ACTION=="unbind", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x030200", TEST=="power/control", ATTR{power/control}="on"
```
```bash
sudo vim /etc/modprobe.d/nvidia-pm.conf 
```
```text
options nvidia "NVreg_DynamicPowerManagement=0x02"
```
然后开启 nvidia-persistenced.service 
```bash
sudo systemctl enable nvidia-persistenced.service
```
> 来自archwiki的说法：
> We also need to enable `nvidia-persistenced.service` to avoid the kernel tearing down the device state whenever the NVIDIA device resources are no longer in use. 
## 创建xorg配置文件
先查看双显卡的信息
```bash
lspci | grep VGA
```
创建xorg配置文件，默认会使用I卡
```bash
sudo vim /etc/X11/xorg.conf
```
```text
Section "ServerLayout"
        Identifier "layout"
        Screen 0 "intel"
        Inactive "nvidia"
        Option "AllowNVIDIAGPUScreens"
EndSection

Section "Device"
        Identifier "nvidia"
        Driver "nvidia"
EndSection

Section "Screen"
        Identifier "nvidia"
        Device "nvidia"
EndSection

Section "Device"
        Identifier "intel"
        Driver "modesetting"
        BusID "PCI:0:2:0" #添写 lspci | grep VGA 输出中I卡的信息
EndSection

Section "Screen"
        Identifier "intel"
        Device "intel"
EndSection
```
