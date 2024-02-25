---
title: debian 安装最新 fastfetch
date: 2022-12-13 18:21:49
updated: 2022-12-15 18:17:49
tags: [debian,Linux,bash]
categories: 
    - [Linux,debian]
    - [shell]
keywords: [debian,安装最新release]
description: debian 中脚本安装最新 fastfetch 的release, 还可以安装点其他的deb
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
# debian 安装 fastfetch

> 我的wsl里面是debian， 虽然debian源里面有neofetch，但在我的机器上总是会卡一下，这是我无法忍受的。
> 

> 然后我想起了C写的fastfetch，应该会更快， 之前在archlinux上面倒是很方便 ，直接 `pacman -S fastfetch` 就安装上了，但debian源里面也没有，手动安装若是想满足我的更新欲望，就只能经常从github下载release然后安装，这依然是我无法忍受的。

> 于是我写了两个脚本，自动获取最新的release，并安装，或者更新。

# 安装、更新最新版fastfetch的shell脚本
下载最新release

```bash
#!/bin/bash
# script_name: get-release-latest.sh
tag_name="`wget -qO- -t1 -T2 "https://api.github.com/repos/${project_name}/releases/latest" | jq -r '.tag_name'`"
release_name="`wget -qO- -t1 -T2 "https://api.github.com/repos/${project_name}/releases/latest" | jq -r '.assets[].name' | grep ".deb"`"
release_url="https://github.com/${project_name}/releases/download/${tag_name}/${release_name}"
wget -c ${release_url} -q --show-progress
```
安装deb包，安装后删除deb包
**此脚本只对安装deb包起作用！！！其他的，自行修改**
```bash
#!/bin/bash
# script_name: fastfetch-install.sh
project_name="LinusDierheimer/fastfetch"
echo "installing ${project_name} ......"
source ./get-release-latest.sh
if [ `whoami` != 'root' ];then
        sudo dpkg -i ${release_name}
else
        dpkg -i ${release_name}
fi
rm ${release_name}
echo "install done"
```
# 食用方式
给fastfetch-install.sh个可执行权限，然后运行
```bash
chmod +x fastfetch-install.sh
./fastfetch-install.sh
# 或者
# bash fastfetch-install.sh
```
到这就安装完了

***
想再安装点别的，复制fastfetch-install.sh到新的脚本，然后改掉project_name，执行新的脚本就行

```bash
cp fastfetch-install.sh fastgithub-install.sh
sed -i "s\project_name="LinusDierheimer/fastfetch"\project_name="dotnetcore/FastGithub"\g" fastgithub-install.sh
chmod +x fastgithub-install.sh
./fastgithub-install.sh
```
想一起安装或更新还可以这么弄, 弄个脚本来执行俩安装脚本

```bash
echo "./fastfetch-install.sh" > two-install.sh
echo "./fastgithub-install.sh" >> two-install.sh
chmod +x two-install.sh
./two-install.sh
```
