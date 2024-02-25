---
title: wsl2 使用windows的代理
date: 2022-12-15 18:15:51
updated: 2022-12-15 18:15:51
tags: [wsl2,代理,bash]
categories: 
    - [Linux,wsl2]
    - [shell]
keywords: 代理
description: wsl2 使用 windows 局域网共享的代理
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
# 新增wsl入站规则
打开powershell(管理员)
```posh
New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow
```
之后就不用管是否wsl能ping通windows了，因为windows默认关闭ICMP回显，除非到防火墙处开启
# 脚本设置代理地址及端口
```bash
#!/bin/bash
# script-name: proxy-set.sh
# 获取网关地址
host_ip="`ip route | grep "default" | awk '{print $3}'`"
# 代理工具的端口
# v2raya port
socks5_port="20170"
http_port="20171"
# 设置代理环境变量
function proxy_on(){
    export ALL_PROXY=socks5://${host_ip}:${socks5_port}
    export http_proxy=http://${host_ip}:${http_port}
    export https_proxy=http://${host_ip}:${http_port}
    export ftp_proxy=http://${host_ip}:${http_port}
    echo "已开启代理"
}
# 删除代理环境变量
function proxy_off(){
    unset ALL_PROXY
    unset http_proxy
    unset https_proxy
    echo "已关闭代理"
}
```
# 食用方法
进入wsl
如果你用的是 zsh 而不是 bash 的话就是写入到 `.zshrc` 里面
```bash
echo "source proxy-set.sh" >> .bashrc
```
然后重新打开 wsl; 或者直接 `source proxy-set.sh` ，然后使用以下命令，不过重启的话就不用再source一遍了, 因为用户 shell 是 bash，每次登录都会读取一遍`.bashrc`, 执行里面的内容
```bash
# 开启代理，不过前提是windows那边的代理工具开启了端口分享
proxy_on
# 关闭代理
proxy_off
```