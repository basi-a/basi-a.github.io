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
btrfs su cr /mnt/@snapshots # 这个安装完snapper之后再挂载, 暂时创建完不用挂载
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
删除`/.snapshots`子卷, 据说是这种一点也不`Archlinux`, 当然我也不知道为啥这么说，我是因为跟其他子卷的格式不统一，强迫症受不了
```bash
sudo btrfs su del /.snapshots
sudo mkdir /.snapshots
```
修改/etc/fstab, 将@snapshots子卷挂载到/.snapshots上面，照着同设备的格式抄, 之后挂载
```bash
sudo mount -a
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
## 休眠配置
### 获取参数
获取`swapfile`所在块设备的`UUID`
```bash
sudo blkid | grep /dev/nvme0n1p2 | awk '{print $3}'
```
获取`resume_offset`, btrfs文件系统和其他文件系统上面的swapfile的获取方式不一样
```bash
sudo btrfs inspect-internal map-swapfile -r /swap/swapfile
```
### 修改内核配置
修改`CMDLINE_LINUX`, 里面加上resume和resume_offset, 比如Grub的要改`/etc/default/grub`
```text
...
CMDLINE_LINUX="... resume=UUID=${上面获得的UUID} resume_offset=${上面获得的resume_offset}"
...
```
如果用的是mkinitcpio的话，需要在/etc/mkinitcpio.conf, 找到HOOKS, 在filesystems参数后面加上resume；当然如果用的是`dracut`，那就啥都不用改
```text
...
HOOKS=(... filesystems resume ...)
...
```
更新`initramfs`
```bash
sudo mkinitcpio -P
# 或者用dracut的, 不过得看好原本叫啥名字
sudo dracut --force --hostonly --no-hostonly-cmdline /boot/initramfs-linux.img $(uname -r)
sudo dracut --force --no-hostonly /boot/initramfs-linux-fallback.img $(uname -r)
## 当然这个是麻烦一点的，直接重装内核就好了嘛
sudo pacman -S linux
```
---
---
---
我弄完最后的状态
![两个磁盘的子卷](https://cdn.basi-a.top/images/2023-12-25-121830_1920x1080_scrot.webp)
