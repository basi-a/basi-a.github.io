---
title: top各个字段的含义
date: 2023-08-07 15:17:05
updated: 2023-08-07 15:17:05
tags: Linux
categories: Linux
keywords: Linux, top
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
以前很多时候只管用，用完就扔，不注意细节，最近因为一些事情意识到了这样不行，于是在这里整理一下`top`命令的各个字段的含义
# TOP
## 什么是top
在Linux和Unix操作系统中，top是一个常用的命令行实用程序，用于动态监控系统中运行的进程和系统资源的使用情况。
## 各个字段含义
|字段名|含义|
|---|---|
|PID (Process ID)|进程ID，唯一标识一个正在运行的进程|
|USER|进程的拥有者，即运行该进程的用户|
|PR (Priority)|进程的优先级。数值越低表示优先级越高|
|NI (Nice value)|进程的Nice值，表示进程的静态优先级调整。数值越高表示优先级越低|
|VIRT (Virtual Memory)|进程使用的虚拟内存大小，包括进程代码、数据和共享库等|
|RES (Resident Memory)|进程使用的实际物理内存大小，即驻留集大小|
|SHR (Shared Memory)|进程使用的共享内存大小|
|S (Status)|进程的状态，可能的状态有：R（运行）、S（睡眠）、D（不可中断的睡眠）、Z（僵尸进程）等|
|%CPU (CPU Usage)|进程在最近一次采样间隔中使用的CPU资源百分比|
|%MEM (Memory Usage)|进程使用的物理内存占系统总内存的百分比|
|TIME+ (CPU Time)|进程的累计CPU占用时间|
|COMMAND (Command)|启动进程的命令行命令或者进程的名称|
