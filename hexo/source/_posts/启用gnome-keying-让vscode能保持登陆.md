---
title: 启用gnome-keying 让vscode能保持登陆
date: 2023-01-08 09:56:14
updated: 2023-01-08 09:56:14
tags: [ArchLinux,Linux]
categories: [Linux,ArchLinux]
keywords: [启用gnome-keying 让vscode能保持登陆]
description: 启用gnome-keying 让vscode能保持登陆
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
在使用dwm之类的窗口管理器，之前也没装过gnome这类的桌面环境，在一个新鲜的环境下,vscode 会去使用gnome-keying来存储登陆信息，而又因为环境中没有gnome-keying，所以vscode每次打开都要重新登陆; 因此本文记录一下，我配置gnome-keying，并登陆自启的过程。
# 安装gnome-keying
```bash
sudo pacman -S gnome-keyring libsecret
# libsecret授予其他应用程序访问密钥环的权限
```
可选的GUI管理工具
```bash
sudo pacman -S seahorse
```
# PAM 自动解锁密钥环
编辑 `/etc/pam.d/login` 文件，在 `auth` 部分的末尾添加 `auth optional pam_gnome_keyring.so`，
在 `session` 末尾添加 `session optional pam_gnome_keyring.so auto_start`
> #%PAM-1.0
>
>auth       required     pam_securetty.so
>auth       requisite    pam_nologin.so
>auth       include      system-local-login
>**auth       optional     pam_gnome_keyring.so**
>account    include      system-local-login
>session    include      system-local-login
>**session    optional     pam_gnome_keyring.so auto_start**
>password   include      system-local-login
# 让gnome-keyring-secrets等自动启动
```bash
cp /etc/xdg/autostart/{gnome-keyring-secrets.desktop,gnome-keyring-ssh.desktop} ~/.config/autostart/
sed -i '/^OnlyShowIn.*$/d' ~/.config/autostart/gnome-keyring-secrets.desktop
sed -i '/^OnlyShowIn.*$/d' ~/.config/autostart/gnome-keyring-ssh.desktop
```
