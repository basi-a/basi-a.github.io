---
title: centos7 安装 zabbix
date: 2022-12-16 17:56:43
updated: 2022-12-16 17:56:43
tags: [centos7,Linux,shell]
categories:
    - [Linux,centos]
    - [shell]
keywords: [centos7,zabbix]
description: centos7 安装 zabbix
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
这是之前做学校实验时我写的zabbix部署脚本
# 网络拓扑
网络拓扑很简单，同一局域网的两台机器，一个是zabbix服务端，一个是被监控端
| IP地址      | 描述 |
| ----------- | ----------- |
| 192.168.200.222 | zabbix服务端     |
| 192.168.200.111 | 部署着其他应用的被监控端  |
# 部署脚本
## 服务端
```bash
#!/bin/bash
# Author        : basi-a
# FIlename      : zabbix-server-install.sh
# Description   : zabbix server install script, by install lamp then install zabbix-server
# mariadb root 用户密码
MARIADB_ROOT_PASSWD=123456
# 数据库 zabbix 用户的密码
ZABBIX_PASSWORD=basi-a@123
# 安装LAMP
function LAMP_install(){
    yum -y install httpd \
    mariadb-server mariadb \
    php
}
# 启动mariadb并初始化
function mariadb_run(){
    systemctl enable mariadb && systemctl start mariadb
    mysqladmin password ${MARIADB_ROOT_PASSWD}
}
# 启动zabbix-server
function zabbix_run(){
    systemctl start zabbix-server zabbix-agent httpd
    systemctl enable zabbix-server zabbix-agent httpd
}

# 安装zabbix
function zabbix_install(){
    rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
    yum clean all
    yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-agent
    # 创建zabbix数据库和zabbix用户
    mysql -uroot -p${MARIADB_ROOT_PASSWD} -e "create database zabbix character set utf8 collate utf8_bin;"
    mysql -uroot -p${MARIADB_ROOT_PASSWD} -e "create user zabbix@localhost identified by '${ZABBIX_PASSWORD}';"
    mysql -uroot -p${MARIADB_ROOT_PASSWD} -e "grant all privileges on zabbix.* to zabbix@localhost;"
    # 导入初始数据
    zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p${ZABBIX_PASSWORD} zabbix
    # 编辑/etc/zabbix/zabbix_server.conf 写入zabbix数据库密码
    sed -i "/^# DBPassword=/a\DBPassword=${ZABBIX_PASSWORD}" /etc/zabbix/zabbix_server.conf
    # 编辑/etc/httpd/conf.d/zabbix.conf 修改时区为亚洲上海
    sed -i "s\# php_value date.timezone Europe/Riga\php_value date.timezone Asia/Shanghai\g" /etc/httpd/conf.d/zabbix.conf
}

function main(){
    systemctl disable firewalld >/dev/null 2>&1
    systemctl stop firewalld >/dev/null 2>1&
    setenforcing 0 >/dev/null 2>&1
    LAMP_install
    mariadb_run
    zabbix_install
    zabbix_run
    echo "visit zabbix with http://server's_IP/zabbix"
    echo "default username: Admin"
    echo "default password: zabbix"
}
main
```
之后就访问 http://服务端IP/zabbix 安装zabbix
然后就可以设置被控端监控项
## 被监控端
```bash
#!/bin/bash
# Author        : basi-a
# FIlename      : zabbix-client-install.sh
# Description   : zabbix client install script, by install zabbix-agent
ZABBIX_SERVER=192.168.200.222
CLIENT_HOSTNAME=192.168.200.111
# 安装zabbix
function zabbix_client_install(){
    rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
    yum clean all
    yum -y install zabbix-agent
    # 编辑/etc/zabbix/zabbix_agentd.conf 指定zabbix-server和本机hostname
    sed -i "s\Server=127.0.0.1\Server=${ZABBIX_SERVER}\g" /etc/zabbix/zabbix_agentd.conf
    sed -i "s\ServerActive=127.0.0.1\ServerActive=${ZABBIX_SERVER}\g" /etc/zabbix/zabbix_agentd.conf
    sed -i "s\Hostname=Zabbix server\Hostname=${CLIENT_HOSTNAME}\g" /etc/zabbix/zabbix_agentd.conf
}
# 启动zabbix-agent
function zabbix_client_run(){
    systemctl enable zabbix-agent && systemctl start zabbix-agent
}

function main(){
    zabbix_client_install
    zabbix_client_run
    echo "zabbix client is OK, you can add it on zabbix server"
}
main
```
被控端, 安装好zabbix-agent, zabbix-server就能进行监控了