---
title: jumpserver尝鲜
date: 2023-10-14 11:34:51
updated: 2023-10-14 11:34:51
tags: [Linux, jumpserver]
categories: Linux
keywords: [Linux, jumpserver]
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
# 什么是JumpServer

> [JumpServer](https://jumpserver.org) 是广受欢迎的开源堡垒机，是符合 4A 规范的专业运维安全审计系统。JumpServer 帮助企业以更安全的方式管控和登录所有类型的资产，实现事前授权、事中监察、事后审计，满足等保合规要求。

JumpServer 堡垒机支持的资产类型包括：

- SSH (Linux / Unix / 网络设备 等)
- Windows (Web 方式连接 / 原生 RDP 连接)
- 数据库 (MySQL / MariaDB / Oracle / SQLServer / PostgreSQL / ClickHouse 等)
- NoSQL (Redis / MongoDB 等)
- GPT (ChatGPT 等)
- 云服务 (Kubernetes / VMware vSphere 等)
- Web 站点 (各类系统的 Web 管理后台)
- 应用 (通过 Remote App 连接各类应用)



# 实验环境规划

此次安装`JumpServer`版本v3.7.2；各个服务器均为`centos7.9`

|     IP地址     |             用途              |
| :------------: | :---------------------------: |
| 192.168.250.10 |       NFS、MySQL、Redis       |
| 192.168.250.11 |       JumpServer node01       |
| 192.168.250.12 |       JumpServer node02       |
| 192.168.250.13 | HAProxy、MiniO、Elasticsearch |

# 密码密钥规划

|     服务      |   用户名   |       密码       |
| :-----------: | :--------: | :--------------: |
|    HAproxy    |   admin    | haproxY_passw0rd |
|     MySQL     | jumpserver | JumpSerVER_Pswd  |
|               |    root    |    Pass_W0Rd     |
|     Redis     |            |  red1s_Passw0rd  |
|     MinIO     |   minio    |    m1n10_pAss    |
| Elasticsearch |  elastic   |  esSeaRch_pswd   |

| JumpServer配置项 |                  值                   |
| :--------------: | :-----------------------------------: |
|    SECRET_KEY    | FtuiouGOIygOIYGIPYvfutoFfyIpiIGvuvTGP |
| BOOTSTRAP_TOKEN  |         GYoFtuOtcYovOyvOTUcFT         |

# 部署服务

## NMR

### NFS

#### 安装epel源，然后安装组件以及依赖

```bash
yum -y install epel-release
yum -y install nfs-utils rpcbind
```

#### 启动NFS服务

```bash
systemctl enable rpcbind nfs-server nfs-lock nfs-idmap --now
```

![image-20231014142627365](https://cdn.basi-a.top/images/image-20231014142627365.png)

#### 配置防火墙

```bash
firewall-cmd --add-service=nfs --permanent --zone=public
firewall-cmd --add-service=mountd --permanent --zone=public
firewall-cmd --add-service=rpc-bind --permanent --zone=public
firewall-cmd --reload
```

#### 配置NFS，并使其生效

```bash
mkdir /data
chmod 777 -R /data

vi /etc/exports
```

下面的是`/etc/exports`的内容

```text
/data 192.168.250.*(rw,sync,all_squash,anonuid=0,anongid=0)
```

打开所有NFS目录共享

```bash
exportfs -a
```

![image-20231014143637894](https://cdn.basi-a.top/images/image-20231014143637894.png)

### MySQL

#### 安装MySQL

```bash
yum -y localinstall http://mirrors.ustc.edu.cn/mysql-repo/mysql57-community-release-el7.rpm
sed -i.bak "s|gpgcheck=1|gpgcheck=0|g" /etc/yum.repos.d/mysql-community.repo 
yum install -y mysql-community-server
```

#### 配置MySQL

```bash
if [ ! "$(cat /usr/bin/mysqld_pre_systemd | grep -v ^\# | grep initialize-insecure )" ]; then
    sed -i "s@--initialize @--initialize-insecure @g" /usr/bin/mysqld_pre_systemd
fi
```

修改`/etc/my.cnf`

```text
[mysqld]
basedir  = /usr/
datadir  = /var/lib/mysql
pid-file = /var/run/mysqld/mysqld.pid
socket   = /var/lib/mysql/mysql.sock
port     = 3306
user     = mysql

log_error                = /var/lib/mysql/mysql-error.log
slow-query-log-file      = /var/lib/mysql/mysql-slow.log
log_bin                  = /var/lib/mysql/mysql-bin.log
relay-log                = /var/lib/mysql/mysql-relay-bin

server-id                = 1
# read_only              = 1
innodb_buffer_pool_size  = 1024M
innodb_log_buffer_size   = 16M
# key_buffer_size        = 64M
key_buffer_size          = 128M
query_cache_size         = 256M
tmp_table_size           = 128M

# lower_case_table_names = 1
binlog_format            = mixed
# binlog_format          = statement
skip-external-locking
skip-name-resolve
character-set-server     = utf8
collation-server         = utf8_bin
# collation-server       = utf8_general_ci
max_allowed_packet       = 16M
thread_cache_size        = 256
table_open_cache         = 4096
back_log                 = 1024
max_connect_errors       = 100000
# wait_timeout           = 864000

interactive_timeout      = 1800
wait_timeout             = 1800

max_connections          = 2048
sort_buffer_size         = 16M
join_buffer_size         = 4M
read_buffer_size         = 4M
# read_rnd_buffer_size   = 8M
read_rnd_buffer_size     = 16M
binlog_cache_size        = 2M
thread_stack             = 192K

max_heap_table_size      = 128M
myisam_sort_buffer_size  = 128M
bulk_insert_buffer_size  = 256M
open_files_limit         = 65535
query_cache_limit        = 2M
slow-query-log
long_query_time          = 2

expire_logs_days         = 3
max_binlog_size          = 1000M
slave_parallel_workers   = 4
log-slave-updates
# slave-skip-errors      = 1062,1053,1146,1032

binlog_ignore_db               = mysql
replicate_wild_ignore_table    = mysql.%
sync_binlog                    = 1

innodb_file_per_table          = 1
innodb_flush_method            = O_DIRECT
innodb_buffer_pool_instances   = 4
innodb_large_prefix            = ON
innodb_log_file_size           = 512M
innodb_log_files_in_group      = 3
innodb_open_files              = 4000
innodb_read_io_threads         = 8
innodb_write_io_threads        = 8
innodb_thread_concurrency      = 8
innodb_io_capacity             = 2000
innodb_io_capacity_max         = 6000
innodb_lru_scan_depth          = 2000
innodb_max_dirty_pages_pct     = 85
innodb_flush_log_at_trx_commit = 2
sql_mode = NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES


[mysqldump]
quick
quote-names
max_allowed_packet = 16M

[client]
default-character-set = utf8

[mysql]
default-character-set = utf8

[isamchk]
key_buffer       = 128M
sort_buffer_size = 4M
read_buffer      = 2M
write_buffer     = 2M

[myisamchk]
key_buffer       = 128M
sort_buffer_size = 4M
read_buffer      = 2M
write_buffer     = 2M
```

#### 启动MySQL，并配置数据库授权

```bash
systemctl enable mysqld --now
vi jumpserver.sql
mysqladmin -uroot -p password Pass_W0Rd
mysql -uroot -pPass_W0Rd -e "source jumpserver.sql"
```

以下是`jumpserver.sql`内容

```sql
create database jumpserver default charset 'utf8';
set global validate_password_policy=LOW;
create user 'jumpserver'@'%' identified by 'JumpSerVER_Pswd';
grant all on jumpserver.* to 'jumpserver'@'%';
flush privileges;
```

之后查看一下执行情况

```bash
mysql -ujumpserver -pJumpSerVER_Pswd -e "show databases;"
```

![image-20231014152240030](https://cdn.basi-a.top/images/image-20231014152240030.png)

#### 配置防火墙

```bash
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.250.0/24" port protocol="tcp" port="3306" accept"
firewall-cmd --reload
```

![image-20231014152427704](https://cdn.basi-a.top/images/image-20231014152427704.png)

### Redis

#### 安装ius源

> [IUS](https://ius.io/)（Inline with Upstream Stable）是一个社区项目，它旨在为Linux企业发行版提供可选软件的最新版RPM软件包。

```bash
yum -y install https://mirrors.aliyun.com/ius/ius-release-el7.rpm
```

#### 安装Redis

```bash
yum install -y redis6
```

![image-20231014172149154](https://cdn.basi-a.top/images/image-20231014172149154.png)

#### 配置Redis

```bash
cp /etc/redis/redis.conf /etc/redis/reids.conf.bak
sed -i "s|bind 127.0.0.1|bind 0.0.0.0|g" /etc/redis/redis.conf
sed -i "s|daemonize no|daemonize yes|g" /etc/redis/redis.conf
sed -i "s|# supervised auto|supervised auto|g" /etc/redis/redis.conf
sed -i '/^protected-mode/a\requirepass red1s_Passw0rd' /etc/redis/redis.conf
sed -i '/^# maxmemory-policy noeviction/a\maxmemory-policy allkeys-lru' /etc/redis/redis.conf
```

#### 启动Redis

```bash
systemctl enable redis --now
```

![image-20231014174710443](https://cdn.basi-a.top/images/image-20231014174710443.png)

#### 配置防火墙

```bassh
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.250.0/24" port protocol="tcp" port="6379" accept"
firewall-cmd --reload
```

### 检查防火墙规则

```bash
firewall-cmd --zone=public --list-all
```

![image-20231014175343155](https://cdn.basi-a.top/images/image-20231014175343155.png)

## JumpServer

### node01

#### 配置NFS

##### 安装NFS依赖，查看NFS服务器情况

```bash
yum -y install nfs-utils
showmount -e 192.168.250.10
```

![image-20231014180735583](https://cdn.basi-a.top/images/image-20231014180735583.png)

##### 挂载NFS目录，并配置开机自动挂载

> 将 Core 持久化目录挂载到 NFS, 默认 /opt/jumpserver/core/data, 请根据实际情况修改
>
> JumpServer 持久化目录定义相关参数为 VOLUME_DIR, 在安装 JumpServer 过程中会提示
>
> 可以写入到 /etc/fstab, 重启自动挂载. 注意: 设置后如果 nfs 损坏或者无法连接该服务器将无法启动

```bash
mkdir -p /opt/jumpserver/core/data
mount -t nfs 192.168.250.10:/data /opt/jumpserver/core/data
echo "192.168.250.10:/data /opt/jumpserver/core/data nfs defaults 0 0" >> /etc/fstab
```

#### 升级Linux内核

就不编译内核了，太慢了，用[elrepo](http://elrepo.org/tiki/kernel-lt)的

```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum -y install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-lt-devel kernel-lt -y
grub2-set-default 0
reboot
```

![image-20231015105624932](https://cdn.basi-a.top/images/image-20231015105624932.png)

#### 安装JumpServer

##### 先下载离线安装包并上传到/opt，然后修改临时配置文件

```bash
cd /opt
tar -zxvf jumpserver-offline-installer-v3.7.2-amd64.tar.gz
cd jumpserver-offline-installer-v3.7.2-amd64
vi config-example.txt
```

需要修改`config-example.txt`内容如下

```text
# 修改下面选项, 其他保持默认, 请勿直接复制此处内容
### 注意: SECRET_KEY 和要其他 JumpServer 服务器一致, 加密的数据将无法解密

################################## 镜像配置 ###################################
#
# 国内连接 docker.io 会超时或下载速度较慢, 开启此选项使用华为云镜像加速
# 取代旧版本 DOCKER_IMAGE_PREFIX
#
DOCKER_IMAGE_MIRROR=1

# 安装配置
### 注意持久化目录 VOLUME_DIR, 如果上面 NFS 挂载其他目录, 此处也要修改. 如: NFS 挂载到 /data/jumpserver/core/data, 则 VOLUME_DIR=/data/jumpserver
VOLUME_DIR=/opt/jumpserver


# Core 配置
### 启动后不能再修改，否则密码等等信息无法解密, 请勿直接复制下面的字符串
SECRET_KEY=FtuiouGOIygOIYGIPYvfutoFfyIpiIGvuvTGP                 # 要其他 JumpServer 服务器一致 (*)
BOOTSTRAP_TOKEN=GYoFtuOtcYovOyvOTUcFT                            # 要其他 JumpServer 服务器一致 (*)
LOG_LEVEL=ERROR                                                  # 日志等级

# JumpServer 容器使用的网段, 请勿与现有的网络冲突, 根据实际情况自行修改
#
DOCKER_SUBNET=172.16.50.0/24

# SESSION_COOKIE_AGE=86400
SESSION_EXPIRE_AT_BROWSER_CLOSE=True                             # 关闭浏览器 session 过期

# MySQL 配置

DB_HOST=192.168.250.10
DB_PORT=3306
DB_USER=jumpserver
DB_PASSWORD=JumpSerVER_Pswd
DB_NAME=jumpserver

# Redis 配置

REDIS_HOST=192.168.250.10
REDIS_PORT=6379
REDIS_PASSWORD=red1s_Passw0rd

# KoKo Lion 配置
SHARE_ROOM_TYPE=redis                                            # KoKo Lion 使用 redis 共享
REUSE_CONNECTION=false                                           # Koko 禁用连接复用
```

##### 执行脚本，安装服务

```bash
./jmsctl.sh install
```

之后安装过程的选项，emmm，其实一路回车就好，预配置文件里面已经写了，这样就一路用写配置文件里面的

##### 启动JumpServer

```bash
./jmsctl.sh start
```

### node02

过程以及配置和node01一样



## HME

### HAproxy

#### 安装epel源

```bash
yum -y install epel-release
```

#### 安装HAproxy，修改配置文件

```bash
yum install -y haproxy
cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
vi /etc/haproxy/haproxy.cfg
```

配置文件内容如下

```text
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

listen stats
    bind *:8080
    mode http
    stats enable
    stats uri /haproxy                      # 监控页面, 请自行修改. 访问地址为 http://192.168.250.13:8080/haproxy
    stats refresh 5s
    stats realm haproxy-status
    stats auth admin:haproxY_passw0rd       # 账户密码, 请自行修改. 访问 http://192.168.250.13:8080/haproxy 会要求输入

#---------------------------------------------------------------------
# check  检活参数说明
# inter  间隔时间, 单位: 毫秒
# rise   连续成功的次数, 单位: 次
# fall   连续失败的次数, 单位: 次
# 例: inter 2s rise 2 fall 3
# 表示 2 秒检查一次状态, 连续成功 2 次服务正常, 连续失败 3 次服务异常
#
# server 服务参数说明
# server 192.168.250.11 192.168.250.11:80 weight 1 cookie web01
# 第一个 192.168.250.11 做为页面展示的标识, 可以修改为其他任意字符串
# 第二个 192.168.250.11:80 是实际的后端服务端口
# weight 为权重, 多节点时按照权重进行负载均衡
# cookie 用户侧的 cookie 会包含此标识, 便于区分当前访问的后端节点
# 例: server db01 192.168.250.11:3306 weight 1 cookie db_01
#---------------------------------------------------------------------

listen jms-web
    bind *:80                               # 监听 80 端口
    mode http

    # redirect scheme https if !{ ssl_fc }  # 重定向到 https
    # bind *:443 ssl crt /opt/ssl.pem       # https 设置

    option httpchk GET /api/health/         # Core 检活接口

    stick-table type ip size 200k expire 30m
    stick on src

    balance leastconn
    server 192.168.250.11 192.168.250.11:80 weight 1 cookie web01 check inter 2s rise 2 fall 3  # JumpServer 服务器
    server 192.168.250.12 192.168.250.12:80 weight 1 cookie web02 check inter 2s rise 2 fall 3

listen jms-ssh
    bind *:2222
    mode tcp

    option tcp-check

    fullconn 500
    balance source
    server 192.168.250.11 192.168.250.11:2222 weight 1 check inter 2s rise 2 fall 3 send-proxy
    server 192.168.250.12 192.168.250.12:2222 weight 1 check inter 2s rise 2 fall 3 send-proxy

listen jms-koko
    mode http

    option httpclose
    option forwardfor
    option httpchk GET /koko/health/ HTTP/1.1\r\nHost:\ 192.168.250.13  # KoKo 检活接口, host 填写 HAProxy 的 ip 地址

    cookie SERVERID insert indirect
    hash-type consistent
    fullconn 500
    balance leastconn
    server 192.168.250.11 192.168.250.11:80 weight 1 cookie web01 check inter 2s rise 2 fall 3
    server 192.168.250.12 192.168.250.12:80 weight 1 cookie web02 check inter 2s rise 2 fall 3

listen jms-lion
    mode http

    option httpclose
    option forwardfor
    option httpchk GET /lion/health/ HTTP/1.1\r\nHost:\ 192.168.250.13  # Lion 检活接口, host 填写 HAProxy 的 ip 地址

    cookie SERVERID insert indirect
    hash-type consistent
    fullconn 500
    balance leastconn
    server 192.168.250.11 192.168.250.11:80 weight 1 cookie web01 check inter 2s rise 2 fall 3
    server 192.168.250.12 192.168.250.12:80 weight 1 cookie web02 check inter 2s rise 2 fall 3

listen jms-magnus
    bind *:30000
    mode tcp

    option tcp-check

    fullconn 500
    balance source
    server 192.168.250.11 192.168.250.11:30000 weight 1 check inter 2s rise 2 fall 3 send-proxy
    server 192.168.250.12 192.168.250.12:30000 weight 1 check inter 2s rise 2 fall 3 send-proxy
```

#### 配置SElinux，启动HAproxy

```bash
setsebool -P haproxy_connect_any 1
systemctl enable haproxy --now
```

#### 配置防火墙

```bash
firewall-cmd --permanent --zone=public --add-port=80/tcp
firewall-cmd --permanent --zone=public --add-port=443/tcp
firewall-cmd --permanent --zone=public --add-port=2222/tcp
firewall-cmd --permanent --zone=public --add-port=33060/tcp
firewall-cmd --permanent --zone=public --add-port=33061/tcp
firewall-cmd --reload
```

### MinIO

#### 安装docker

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
yum makecache fast
yum -y install docker-ce
```

配置docker，然后启动docker

```bash
mkdir /etc/docker/
vi /etc/docker/daemon.json
systemctl enable docker --now
```

> `/etc/docker/daemon.json`的内容如下

```text
{
  "live-restore": true,
  "registry-mirrors": ["https://hub-mirror.c.163.com","https://mirror.ccs.tencentyun.com","https://registry.docker-cn.com"],
  "log-driver": "json-file",
  "log-opts": {"max-file": "3", "max-size": "10m"}
}
```

#### 安装MinIO

```bash
mkdir -p /opt/jumpserver/minio
cd /opt/jumpserver/minio
vi docker-compose.yml
docker compose up -d
```

`docker-compose.yml`内容如下

```yaml
version: '3'
services:
    minio:
        container_name: jms_minio
        image: minio/minio:latest
        ports:
            - "9000:9000"
            - "9001:9001"
        volumes:
            - /opt/jumpserver/minio/data:/data
            - /opt/jumpserver/minio/config:/root/.minio
        restart: always
        environment:
            - MINIO_ROOT_USER=minio
            - MINIO_ROOT_PASSWORD=m1n10_pAss
        command: server /data --console-address ":9001"
        healthcheck:
            test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
            interval: 30s
            timeout: 20s
            retries: 3
```

#### 进入MinIO，创建存储桶

![image-20231014212031171](https://cdn.basi-a.top/images/image-20231014212031171.png)

### Elasticsearch

#### 安装Elasticsearch

```bash
mkdir -p /opt/jumpserver/elasticsearch
cd /opt/jumpserver/elasticsearch
vi docker-compose.yml
docker compose up -d
```

`docker-compose.yml`内容如下

```yaml
version: '3'
services:
    es:
        container_name: jms_es
        image: docker.elastic.co/elasticsearch/elasticsearch:7.17.6
        ports:
            - "9200:9200"
            - "9300:9300"
        volumes:
            - /opt/jumpserver/elasticsearch/data:/usr/share/elasticsearch/data
            - /opt/jumpserver/elasticsearch/logs:/usr/share/elasticsearch/logs
        restart: always
        environment:
            - cluster.name=docker-cluster
            - discovery.type=single-node            # 单节点
            - network.host=0.0.0.0                  # 锁定物理内存, 不使用 swap
            - bootstrap.memory_lock=true
            - xpack.security.enabled=true         # 开启安全模块
            - TAKE_FILE_OWNERSHIP=true            # 自动修改挂载文件夹的所属用户
            - ES_JAVA_OPTS=-Xms2g -Xmx2g      # JVM 内存大小, 推荐设置为主机内存的一半
            - ELASTIC_PASSWORD=esSeaRch_pswd        # Elasticsearch 密码
```

检查一下`es`是否可用

```bash
curl -XGET -u elastic http://192.168.250.13:9200
```

![image-20231015143405199](https://cdn.basi-a.top/images/image-20231015143405199.png)

# 食用

## JumpServer中配置 MinIO

> - 访问 JumpServer Web 页面并使用管理员账号进行登录。
> - 系统设置->组件设置->录像存储
> - 根据下方的说明进行填写，保存后在 [终端管理] 页面对所有组件进行 [更新]，录像存储选择 [jms-mino]，提交。
>
> | 选项            | 参考值                      | 说明                   |
> | --------------- | --------------------------- | ---------------------- |
> | 名称 (Name)     | jms-minio                   | 标识, 不可重复         |
> | 类型 (Type)     | Ceph                        | 固定, 不可更改         |
> | 桶名称 (Bucket) | jumpserver                  | Bucket Name            |
> | Access key      | minio                       | MINIO_ROOT_USER        |
> | Secret key      | m1n10_pAss                  | MINIO_ROOT_PASSWORD    |
> | 端点 (Endpoint) | http://192.168.2500.13:9000 | minio 服务访问地址     |
> | 默认存储        |                             | 新组件将自动使用该存储 |

![image-20231015134936369](https://cdn.basi-a.top/images/image-20231015134936369.png)

## JumpServer配置 Elasticsearch

> - 访问 JumpServer Web 页面并使用管理员账号进行登录。
> - 系统设置->组件设置->命令存储
> - 根据下方的说明进行填写，保存后在 [终端管理] 页面对所有组件进行 [更新]，命令存储选择 [jms-es]，提交。
>
> | 选项         | 参考值                                           | 说明                    |
> | ------------ | ------------------------------------------------ | ----------------------- |
> | 名称 (Name)  | jms-es                                           | 标识, 不可重复          |
> | 类型 (Type)  | Elasticsearch                                    | 固定, 不可更改          |
> | 主机 (Hosts) | http://elastic:esSeaRch_pswd@192.168.250.13:9200 | http://es_host:es_port  |
> | 索引 (Index) | jumpserver                                       | 索引                    |
> | 忽略证书认证 |                                                  | https 自签 ssl 需要勾选 |
> | 默认存储     |                                                  | 新组件将自动使用该存储  |

![image-20231015144559333](https://cdn.basi-a.top/images/image-20231015144559333.png)

## 直接访问HAproxy服务器的IP，愉快食用

![image-20231015151431511](https://cdn.basi-a.top/images/image-20231015151431511.png)