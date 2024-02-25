---
title: docker-compose mariadb+redis 简单部署
date: 2023-01-27 13:12:41
updated: 2023-01-27 13:12:41
tags: [docker, docker-compose]
categories: [docker, docker-compose]
keywords: 
description: "使用docker-compose 来简单部署mariadb和redis"
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
# 安装docker和docker-compose
安装这个很简单，就不写了。。。。
## 添加用户到docker组
把操作`docker`的用户，添加到`docker`组，这样就不用每次都要切root或者输入sudo密码（啥都用root用户是一种愚蠢的行为。。。小声）
```bash
sudo usermod -a -G docker username
```
# 新建文件夹，创建docker-compose.yml 文件
```bash
mkdir test
cd test && vim docker-compose.yml
```
## docker-compose.yml 内容
```yml
# Use root/123456 as user/password for mariadb
# adminer is a simple tool to control mariadb
# Use 123456 as redis password
version: '3.1'

services:
  mariadb:
    image: mariadb
    container_name: db
    restart: always
    ports:
      - "3306:3306"
    volumes:
      - ./mariadb/data:/var/lib/mysql:rw
    environment:
      MARIADB_ROOT_PASSWORD: "123456"
  adminer:
    image: adminer
    container_name: adminer
    restart: always
    ports:
      - 8080:8080
  redis:
    image: redis
    container_name: redis
    restart: always
    volumes:
      - ./redis/data:/data:rw
      - ./redis/conf/redis.conf:/usr/local/etc/redis/redis.conf:rw
    command: redis-server /usr/local/etc/redis/redis.conf
    ports:
      - 6379:6379
```
## 创建 redis.conf
```bash
mkdir -p redis/conf
vim redis/conf/redis.conf
```
配置文件内容
```text
protected-mode no
port 6379
timeout 0
save 900 1
save 300 10
save 60 10000
rdbcompression yes
dbfilename dump.rdb
dir /data
appendonly yes
appendfsync everysec
requirepass 123456
```
## 在docker-compose.yml所在目录启动docker
```bash
docker-compose up -d
```
如此便可以后台运行`mariadb,redis 和adminer`了。

>**如果报错`ipv4 ip_forward`相关的错误**;
>
>写入`net.ipv4.ip_forward=1`到`/etc/sysctl.d/ipv4-forward.conf`，然后重启就好
