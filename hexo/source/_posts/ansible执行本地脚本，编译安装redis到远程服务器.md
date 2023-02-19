---
title: ansible执行本地脚本，编译安装redis到远程服务器
date: 2023-02-18 15:38:01
updated: 2023-02-18 15:38:01
tags: 
  - ansible
  - centos
  - archlinux
categories: 
  - [Linux,centos]
  - [Linux,ArchLinux]
keywords:  ansible执行本地脚本，编译安装redis到远程服务器
description: ansible执行本地脚本，编译安装redis到远程服务器
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
# 配置ansible
我本地的机器是archlinux, 先安装ansible
```bash
paru -S ansible
```
配置`/etc/ansible/hosts`, 记录下服务器的IP, 以及普通用户sudo提权密码
```bash
[centos8]
192.168.56.103 ansible_become_pass="sudo_password"
```
发送自己的`ssh`公钥到服务器
```bash
ssh-copy-id -i /home/basi/.ssh/id_rsa.pub basi@192.168.56.103
```
检测`ansible`是否配置正确，返回的结果为`root`则配置正确
```bash
ansible centos8 -u basi -b --become-user root --become-method sudo -m command -a "whoami"
```
# 安装redis
编写安装脚本`redis-install.sh`
```bash
#!/bin/bash
cd /opt
wget https://download.redis.io/redis-stable.tar.gz
tar -zxvf redis-stable.tar.gz
cd redis-stable && make install #默认安装到/usr/local/bin
cp redis.conf /usr/local/etc/redis.conf
nohup redis-server /usr/local/etc/redis.conf >>/var/log/redis.log 2>&1 &
```
执行本地脚本，编译安装redis到服务器
```bash
ansible centos8 -u basi -b --become-user root --become-method sudo -m script -a "/home/basi/redis-install.sh"
```
检查是否安装成功，且在运行中
```bash
ansible centos8 -u basi -b --become-user root --become-method sudo -m shell -a "netstat -tulnp | grep redis-server"
```
成功运行的结果：
>192.168.56.103 | CHANGED | rc=0 >>
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      4528/redis-server 1 
tcp6       0      0 ::1:6379                :::*                    LISTEN      4528/redis-server 1

