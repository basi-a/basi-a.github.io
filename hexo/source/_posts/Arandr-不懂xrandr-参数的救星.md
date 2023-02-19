---
title: Arandr --不懂xrandr 参数的救星
date: 2022-07-03 17:41:10
updated: 2022-12-15 18:10:10
tags: [dwm,suckless]
categories: 
    - [Linux]
keywords: [arandr,xrandr]
description: Arandr --不懂xrandr 参数的救星
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
# 新增显示器
> 一直以来我的 Archlinux 一直使用着 `dwm` 窗口管理器，也没用完整的桌面环境。
> 最近把家里旧显示器给翻出来了，接到我的本子上之后就按默认复制我的主显示器。但正经人谁副屏是不用来扩展屏幕的，于是我便开始想办法改设置，要是用的像kde这种完整的de 我也就不用发愁了，毕竟设置里的功能十分完善,可我用着简陋的dwm，整个dwm都是我自己打补丁编译安装的，想改设置就只能写配置文件，或者用命令了。
# Arandr 设置多显示器输出
> 网络上翻了半天，结果都是用 `xrandr` 这一命令来控制双显示器的输出，可参数我实在看不懂。
> 最后回到了万能的 `archwiki` ,结果发现了 `Arandr` 这个好用的图形化工具。
> 这个工具可以用鼠标拖拽来修改副显示器的用途，并且还可以将设置保存成shell 脚本 。
> 这一功能可太好用了：打开脚本一看还是调用的 `xrandr` ，于是我就把这脚本里的命令直接给复制到我dwm的开机自启脚本 `~/.dwm/autostart.sh` 里了。

# 我的 suckless 全家桶
> 既然提到了 `dwm` 那我就再此贴一下我的suckless全家桶的git仓库吧 ヾ(≧▽≦*)o
> https://gitee.com/basi-a/my_dwm
> https://gitee.com/basi-a/my_st
> https://gitee.com/basi-a/my_slock