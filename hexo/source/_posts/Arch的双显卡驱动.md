---
title: Arch的双显卡驱动
date: 2022-02-09 17:55:36
updated: 2022-12-15 17:55:36
tags: [ArchLinux]
categories: 
    - [Linux,ArchLinux]
keywords: [ArchLinux,双显卡]
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
**注意 ：本文两种方案二选一 ！！**
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
一般来说，装完驱动，不用配置啥，直接`prime-run xxx`启动想用N卡的程序就行；
但也可以/etc/X11/xorg.conf.d/nvidia.conf里面显式的配置一下
```text
Section "ServerLayout"
  Identifier "layout"
  Screen 0 "iGPU"
  Option "AllowNVIDIAGPUScreens"
EndSection

Section "Device"
  Identifier "iGPU"
  Driver "modesetting"
  BusID "PCI:0:2:0"
EndSection

Section "Screen"
  Identifier "iGPU"
  Device "iGPU"
EndSection

Section "Device"
  Identifier "dGPU"
  Driver "nvidia"
EndSection
```
下面的不配也行
## 添加N卡配置文件
> 对于在 Intel Coffee Lake 或更高版本 CPU 以及某些 Ryzen CPU（如 5800H）平台上运行的图灵显卡，可以 在不使用的时候完全关闭 GPU。需要以下 udev 规则：
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
> 我们还需要启用nvidia-persistenced.service服务以避免内核在 NVIDIA 设备资源不再使用时清空设备状态。

其他的配置像`反向prime`我用不到，看[archwiki](https://wiki.archlinux.org/title/PRIME)吧