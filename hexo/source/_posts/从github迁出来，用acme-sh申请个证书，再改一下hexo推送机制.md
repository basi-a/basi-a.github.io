---
title: 从github迁出来，用acme.sh申请个证书，再改一下hexo推送机制y
date: 2024-02-27 16:45:16
updated: 2024-02-27 16:45:16
tags: git
categories: git
keywords: git
description: 从github迁出来，用acme.sh申请个证书，再改一下hexo推送机制
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
# 悲报
> 不知何原因，我的`github`账号被认为有异常登陆，给我重置了随机密码，然后把我所有仓库以及账户全屏蔽了，好在给我发邮件让我改密码了，倒是东西全拿回来了，但是不知道是这个原因还是github有BUG了，workflow和hgit page还是不能用，于是就迁移博客到这个小VPS上了

> 但是呢，老是从腾讯云下载证书，真的好麻烦啊，所有就用acme.sh来自动申请/续签证书。
# acme
`acmesh-official/acme.sh` 这个项目自动申请SSL证书

## install socat
```bash
sudo apt install socat
```
## install acme
```bash
curl https://get.acme.sh | sh
```
## create dnspod token and set env
### 到DNSpod创建token
[dnspod token create](https://console.dnspod.cn/account/token/token)
## 设置环境变量
```bash
basi@VM67811:~$ export DP_Id="xxx"
basi@VM67811:~$ export DP_Key="xxxxxxxxxxxxxxxxxxxx"
```
# 申请证书
```bash
source .bashrc
acme.sh --register-account -m basi-a@outlook.com
acme.sh --issue --dns dns_dp -d basi-a.top -d *.basi-a.top
```
[各种DNS服务商申请方法](https://github.com/acmesh-official/acme.sh/wiki/dnsapi#dns_dp)
## 申请结果
```text
-----END CERTIFICATE-----
[Sun Feb 25 02:11:43 PM UTC 2024] Your cert is in: /root/.acme.sh/basi-a.top_ecc/basi-a.top.cer
[Sun Feb 25 02:11:43 PM UTC 2024] Your cert key is in: /root/.acme.sh/basi-a.top_ecc/basi-a.top.key
[Sun Feb 25 02:11:43 PM UTC 2024] The intermediate CA cert is in: /root/.acme.sh/basi-a.top_ecc/ca.cer
[Sun Feb 25 02:11:43 PM UTC 2024] And the full chain certs is there: /root/.acme.sh/basi-a.top_ecc/fullchain.cer
```
# 给nginx用上申请的证书
## 安装证书
```bash
sudo -i
mkidr /path/to/ssl
acme.sh --installcert -d basi-a.top \
--key-file /path/to/ssl/basi-a.top.key \
--fullchain-file /path/to/ssl/basi-a.top.fullchain.cer \
--reloadcmd "service nginx force-reload"
```
## 给我迁移的静态博客套上SSL
先删掉`/etc/nginx/sites-available/default`, 免得老是链接到`/etc/nginx/sites-enabled/`这里面
```config
server {
        listen 443 ssl;
        listen [::]:443 ssl;
        server_name basi-a.top;
        server_name www.basi-a.top;
        ssl_certificate /path/to/ssl/basi-a.top.fullchain.cer;
        ssl_certificate_key /path/to/ssl/basi-a.top.key;
        root /path/to/xxx;
        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }
}
server {
        listen 80;
        listen [::]:80;

        server_name basi-a.top;
        server_name www.basi-a.top;

        location / {
                try_files $uri $uri/ =404;
                add_header Strict-Transport-Security "max-age=31536000
; includeSubDomains; preload";
        }
}
```
重启或者重载配置就齐活了
# hexo 发布方式修改
## 先搞个git仓库在VPS上吧
这些全切root用户操作吧，方便点
### git用户证书登陆
给git用户添加咱的ssh密钥, 把咱本地的ssh公钥写到`authorized_keys`里面
```bash
useradd git -m
cd /home/git/
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
```
### 仓库初始化
```bash
cd /home
mkdir gitrepo
chown git:git gitrepo/
cd gitrepo
git init --bare hexo.git
chown -R git:git hexo.git
```
### 创建更新事件
```bash
vim /home/gitrepo/hexo.git/hooks/post-receive
chown -R git:git /var/www/public
chmod 700 /home/gitrepo/hexo.git/hooks/post-receive
chown git:git /home/gitrepo/hexo.git/hooks/post-receive
chown -R git:git /var/www/public
```
hooks内容`hexo.git/hooks/post-receive`
```bash
#!/bin/bash

GIT_REPO=/home/gitrepo/hexo.git                      # 空 Git 仓库的文
  夹，触发 hook 时已经存入了内容
TMP_GIT_CLONE=/tmp/hexo                  # 缓存文件夹，存在 /tmp 下可
  随意读写
PUBLIC_WWW=/var/www/public                # 之前创建的 blog 文件夹，用
  网站主目录
rm -rf ${TMP_GIT_CLONE}                  # 删除缓存的全部内容
git clone ${GIT_REPO} ${TMP_GIT_CLONE}       # 将 Git 仓库被上传的内容
  入缓存
rm -rf ${PUBLIC_WWW}/*                   # 删除网站主目录全部内容
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}  # 将缓存目录所有内容复制到主
  录
rm -rf ${PUBLIC_WWW}/.git # 要删掉不该有的东西
```
## 改hexo配置
`vim _config.yaml`
```yaml
deploy:
  type: git
  repo: git@dev.basi-a.top:hexo.git
  # example, https://github.com/hexojs/hexojs.github.io
  branch: master
```

### 推送逝世吧
```bash
hexo clean
hexo g
hexo d
```
