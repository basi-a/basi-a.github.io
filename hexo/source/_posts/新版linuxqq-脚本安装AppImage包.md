---
title: 新版linuxqq 脚本安装AppImage包
date: 2022-12-31 09:40:21
updated: 2022-12-31 09:40:21
tags: [linuxqq,AppImage]
categories: 
    - [Linux]
    - [shell]
keywords: [新版linuxqq,脚本安装,AppImage包]
description: 新版linuxqq 脚本安装AppImage包
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
2022年12月30日，官方新版的[linuxqq v3.0.0](https://im.qq.com/linuxqq/index.shtml)发布，不用再忍受上古画风的linuxqq了
为了方便自己使用中安装更新于是写了个小脚本
# 脚本
能sudo提升权限的用户执行
```bash
#!/bin/bash
DownloadUrl=`curl -s 'https://im.qq.com/rainbow/linuxQQDownload/' -H 'user-agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36' | \
	grep "x64DownloadUrl" | \
	awk -F"\",\"" '{print $7}' | \
	awk -F"\":\"" '{print $2}' | \
	awk -F"\"},\"" '{print $1}'`

install(){
	echo "url: ${DownloadUrl}"
	echo "install start"
	wget ${DownloadUrl} -c -q --show-progress -O linuxqq
	chmod +x linuxqq
	sudo cp linuxqq /usr/local/bin/linuxqq
	rm linuxqq
	echo "install done"
}
main(){
	install
}
main
```
# 结语
虽然官方发布了deb和rpm包，但我不想看着装上不知道是啥的依赖，所以我更喜欢安装appimage的包，至于什么桌面图标的，我用着dwm也看不见，所以脚本中便没有设置这一项的