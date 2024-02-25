---
title: wsl fix bug and init
date: 2023-07-12 17:34:07
updated: 2023-07-12 17:34:07
tags: [wsl2, shell]
categories: [Linux, wsl2]
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
# 前言
最近重新用上了Windows，又开始弄wsl，发现了些小问题，分别是读宿主机的`hosts`和`/usr/lib/wsl/lib`里面本应该是链接但都是文件，于是写了个小脚本解决这个问题
# 脚本
```bash
#!/bin/bash
mkdir /usr/lib/wsl/lib-link
ln -sf /usr/lib/wsl/lib/* /usr/lib/wsl/lib-link
sed -i "s|/usr/lib/wsl/lib|/usr/lib/wsl/lib-link|g" /etc/ld.so.conf.d/ld.wsl.conf

cat >/etc/wsl.conf<<-EOF
[automount]
ldconfig = false
[network]
generateHosts = false
[boot]
systemd = true
EOF
```
