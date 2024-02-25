---
title: ArchLinux的安装
date: 2022-01-15 12:55:00
updated: 2022-12-15 17:51:16
tags: "Linux"
categories:
    - [Linux,ArchLinux]
keywords: "ArchLinux"
description: ArchLinux的安装
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
按照 archwiki 安装

# 安装前的准备

- 一个烧录好archiso的 U盘
- 独立的空硬盘
- 一台支持UEFI的电脑(虚拟机也行，开启UEFI模式)
- 脑子，手

# 验证引导模式，以及连接网络

## 验证引导模式

```bash
ls /sys/firmware/efi/efivars
```

> 如果命令结果显示了目录且没有报告错误，则系统以 UEFI 模式引导。如果目录不存在就是BIOS模式

## 连接网络

### 有线连接或虚拟机

不用管网络问题直接下一步（DHCP自动分配IP地址）

### WIFI

使用 `iwctl` 连接网络

```bash
iwctl
[iwd]# device list //列出无线网卡名称
[iwd]# station device scan //device 写网卡名称
[iwd]# station device get-networks //列出可连接wifw
[iwd]# station device connect SSID //SSID wifi名称
```

# 更新系统时间

保证系统时间的准确

```bash
timedatectl set-ntp true
```

# 硬盘分区

查看硬盘名称， 如：`/dev/sda`  `/dev/nvme0n1`

```bash
fdisk -l
```

使用 `cfdisk` 对硬盘进行分区 以 `/dev/sda` 为例

| 挂载点        | 分区      | 分区类型                  | 建议大小 |
| ------------- | --------- | ------------------------- | -------- |
| /mnt/boot/efi | /dev/sda1 | efi 系统分区              | 至少260M |
| [SWAP]        | /dev/sda2 | LInux swap （交换空间）   | 大于512M |
| /mnt          | /dev/sda3 | Linux x86_64 根目录 （/） | 剩余空间 |

## 格式化分区

### /dev/sda3

根分区建立ext4 文件系统

```bash
mkfs.ext4 /dev/sda3
```

### /dev/sda1

将efi分区格式化为 fat32

```bash
mkfs.fat -F 32 /dev/sda1
```

### /dev/sda2

格式化 swap 分区，并启动swap 分区

```bash
mkswap /dev/sda2
swapon /dev/sda2
```

## 挂载分区

```bash
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

# 选择镜像

根据下载速度进行排序，并筛选出前 20 个最近同步的国内镜像，最后将结果覆写到 `/etc/pacman.d/mirrorlist` 文件内：

```bash
reflector --country China --verbose --latest 20 --sort rate --save /etc/pacman.d/mirrorlist
```

# 安装必须的软件包

```bash
pacstrap /mnt base base-devel linux linux-firmware vi vim networkmanager dhcpcd
```

# 配置系统

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

# chroot 到新系统

```bash
arch-chroot /mnt
```

## 时区

设置时区 (上海时区)

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

## 本地化

编辑 `/etc/locale.gen`，然后取消掉 `en_US.UTF-8 UTF-8` 和其他需要的地区前的注释（#）。

接着执行 `locale-gen` 以生成 locale 信息：

```bash
locale-gen
```

然后创建 locale.conf文件，并编辑设定 LANG 变量

`/etc/locale.conf`

写入

```bash
LANG=en_US.UTF-8
```



## 网络配置

创建 `hostname` 文件

`/etc/hostname`

写入

```bash
myhostname #主机名
```

添加对应信息到 `hosts`

`/ect/hosts`

写入

```bash
127.0.0.1	localhost
::1		  localhost
127.0.1.1	myhostname.localdomain myhostname #主机名.本地域名 主机名
```

### 设置NetworkManager 和dhcpcd 开机启动

```bash
systemctl enable NetworkManager && systemctl enable dhcpcd
```

## root密码

设置root密码

```bash
passwd
```

## 安装引导程序

### 微码

如果有AMD或intel的CPU要安装微码

对于 AMD 处理器，安装 `amd-ucode`

对于 Intel 处理器，安装 `intel-ucode`

如果你在一个移动介质上安装Arch Linux，需要应该安装以上两个厂商处理器的微码软件包。

```bash
pacman -S intel-ucode # AMD的CPU就是 amd-ucode
```

### 安装GRUB

```bash
pacman -S grub efibootmgr
```

#### 安装引导程序

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
```

#### 生成引导文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

# 重启

输入 `exit` 或 按 `ctrl+D` 退出chroot环境 

```bash
reboot # 重启
```

**************

到此，arch安装完成，可以进tty了，另外无线网络可以使用`nmtui`重新连接