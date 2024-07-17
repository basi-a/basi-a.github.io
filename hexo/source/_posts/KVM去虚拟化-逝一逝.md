---
title: KVM去虚拟化? 逝一逝
date: 2024-07-17 17:48:42
updated: 2024-07-17 17:48:42
tags: QEMU/KVM
categories: QEMU/KVM
keywords: QEMU/KVM
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
### CPU配置

对于虚拟机的CPU需要配置成host-passthrough模式, 拓扑也要手动设置，不然任务管理器就认为就只是一个大核心

![CPU配置](https://cdn.basi-a.top/images/CPU配置.png)

### VM XML配置

在虚拟机xml中的`<features></features>`中插入

```xml
<kvm>
  <hidden state='on'/>
</kvm>
```

![xml配置1](https://cdn.basi-a.top/images/xml配置1.png)

在`<cpu></cpu>`中插入

```xml
<feature policy='disable' name='hypervisor'/>
```



![XMLCPU配置](https://cdn.basi-a.top/images/xmlCPU配置.png)



### 结果和发癫

下面是结果，应该大部分游戏和有检测的程序都可以了, 嵌套虚拟化也可以了。

当然要是真想玩游戏，还是改改`grub`配置打开`iommu`, 然后`直通PCI设备`，像键鼠、蓝牙、独显，这样才能最好效果。当然virtio是必须的, 要么直通硬盘，要么就用virtio, 这样性能才好。

virtio windows是要驱动的，装系统时候就挂载 [virtio-win.iso](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/) 一起装上就好了。

直通蓝牙的话，需要知道是什么硬件去找驱动才行。就比如我的联发科的网卡带的蓝牙，直通进去之后就随便给我打了一个没用的驱动，还是自己装得，效果很好，xbox 手柄搓着美滋滋。



哦哈哈哈， 现在有种想法，半劈CPU内存，分别直通核显和独显给linux和windows, 接俩显示器，搭配 [synergy-core](https://github.com/symless/synergy-core) 或者硬件kvm switch, 我就可以同时俩系统运行...嘿嘿嘿......  等哪天有资源了用PVE玩玩



![基本配置结果](https://cdn.basi-a.top/images/基础配置结果.png)
