---
title: Install Minikube on Archlinux to learn for fun
date: 2023-05-11 14:49:33
updated: 2023-05-11 14:49:33
tags: kubernetes, ArchLinux , k8s
categories: 
    - [Linux, ArchLinux]
    - [kubernetes]
keywords: minikube, Minikube
description: Install Minikube to learch k8s
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
# Minikube
> minikube quickly sets up a local Kubernetes cluster on macOS, Linux, and Windows. We proudly focus on helping application developers and new Kubernetes users.
[minikube docs](https://minikube.sigs.k8s.io/docs/)

# Archlinux 安装
前提是有`docker`环境，或者`KVM`，首选`docker`, 其他的需要改配置文件
```bash
sudo pacman -S minikube kubectl
```
# 启动
```bash
#首次启动，要拉取镜像，创建容器很慢，指定cn源
minikube start --image-mirror-country='cn'
#之后有了容器和镜像就不用指定镜像了
#minikube start
```
# 检查pod状态
```bash
kubectl get po -A
```

# 打开dashboard
```bash
minikube dashboard
```
之后就可以愉快的玩耍单机部署的k8s了
