---
title: 终于用上了不敢用的btrfs，嗯真香
date: 2023-12-24 22:18:07
updated: 2023-12-24 22:18:07
tags: filesystem
categories: Linux
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
# 系统环境
我的新机器有两块1T大小的SSD, 一块 `/dev/sda` 一块 `/dev/nvme0n1`，分区情况如下

|磁盘/分区|文件系统|
|---|---|
|/dev/sda|---|
|/dev/sda1|btrfs|
|/dev/nvme0n1|---|
|/dev/nvme0n1p1|FAT32|
|/dev/nvme0n1p2|btrfs|

`btrfs`子卷创建情况如下
|分区|子卷|挂载点|
|---|---|---|
|/dev/nvme0n1p2|@|/|
|/dev/nvme0n1p2|@log|/var/log|
|/dev/nvme0n1p2|@cache|/var/cache|
|/dev/nvme0n1p2|@snapshots|/.snapshots|
|/dev/sda1|@home|/home|
|/dev/sda1|@swap|/swap|

# 格式化分区
```bash
mkfs.fat -F 32 /dev/nvme0n1p1
mkfs.btrfs /dev/sda1
mkfs.btrfs /dev/nvme0n1p2
```
# 挂载父卷，创建子卷
```bash
mount /dev/nvme0n1p2 /mnt
btrfs su cr /mnt/@
btrfs su cr /mnt/@log
btrfs su cr /mnt/@cache
btrfs su cr /mnt/@snapshots
umount /mnt
mount /dev/sda1 /mnt
btrfs su cr /mnt/@home
btrfs su cr /mnt/@swap
umount /mnt
```
# 挂载子卷, 以及引导分区
```bash
mount /dev/nvme0n1p2 /mnt -o subvol=@,noatime,discard=async,compress=zstd
mount /dev/nvme0n1p2 /mnt/var/log -o subvol=@log,noatime,discard=async,compress=zstd
mount /dev/nvme0n1p2 /mnt/var/cache -o subvol=@cache,noatime,discard=async,compress=zstd
mount /dev/sda1 /mnt/home -o subvol=@home,noatime,discard=async,compress=zstd
mkdir /mnt/swap
mount /dev/sda1 /mnt/swap -o subvol=@swap,noatime,discard=async,compress=zstd
mount /dev/nvme0n1p1 /boot
```
# 安装系统，生成新系统`/etc/fstab`
## 安装系统
安装基础包
```bash
pacstrap -K /mnt base base-devel linux linux-headers linux-firmware vim amd-ucode btrfs-progs
arch-chroot /mnt
```
`chroot`到新系统环境进行常规安装过程配置，常规操作略

需要在新系统环境修改`/etc/mkinitcpio.conf`, 找到`MODULES=()`, 括号里面写上`btrfs`,使系统启动时加载`btrfs`内核模块，从而正常启动系统

**每次编辑/etc/mkinitcpio.conf，都要记得重新生成initramfs**
```bash
mkinitcpio -P
```
之后，安装`grub`就行了
# 重启进入新系统，用上snapper，为作死保驾护航
安装`snapper`,`snapper-gui`(非必须)
```bash
sudo pacman -S snapper
paru -S snapper-gui
```
安装`grub-btrfs`和`inotify-tools`,开启一个服务, 使得每次出现新的快照时，都会在grub配置文件添加快照入口，在不恢复快照情况下直接进入快照，方便排查
```bash
sudo systemctl enable grub-btrfsd --now
```

创建对`/`进行快照的配置文件
```bash
sudo snapper -c root create-config /
```

编辑配置文件`/etc/snapper/configs/root`, 修改一下，可以进行快照的用户和组
```text
ALLOW_USERS=""
ALLOW_GROUPS="wheel"
```

启动定时任务
```bash
sudo systemctl enable snapper-timeline.timer --now
sudo systemctl enable snapper-cleanup.timer --now
```

之后就可以尽情的折腾了，大不了就回滚

---
---
---
# 禁用写时复制
```bash
sudo chattr +C /var/log
sudo chattr +C /var/cache
```
# 交换文件
```bash
sudo btrfs filesystem mkswapfile --size 32g --uuid clear /swap/swapfile
swapon /swap/swapfile
```
写入`/swap/swapfile none swap defaults 0 0`到`/etc/fstab`

我弄完最后的状态
![两个磁盘的子卷](https://cdn.basi-a.top/images/2023-12-25-121830_1920x1080_scrot.webp)
