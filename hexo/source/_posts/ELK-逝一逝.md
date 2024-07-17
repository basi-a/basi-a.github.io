---
title: ELK? 逝一逝
date: 2024-07-17 17:48:30
updated: 2024-07-17 17:48:30
tags: ELK
categories: ELK
keywords: ELK
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
# 下载组件

## 核心组件

ELK的核心组件是 [Elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)  [Kibana](https://www.elastic.co/cn/downloads/kibana) 这两个，就直接下载tar包吧, 我喜欢通用性的

## 数据收集组件

Agent收集方案有三种, [Elastic Agent](https://www.elastic.co/cn/downloads/elastic-agent)、[Beats](https://www.elastic.co/cn/downloads/beats)、[Logstatsh](https://www.elastic.co/cn/downloads/logstash) , 其中Beats每种数据收集都分成一个Agent，logstash只能收集日志，而Elastic Agent则是单个Agent收集最为全面的。

所以就选用Elastic Agent吧

我就根据习惯全解压缩放到`$HOME/ELK`里面吧, 先把核心装好，其他再说

# 配置环境变量并启动

## Elasticsearch

启动es

```bash
cd $HOME/ELK/elasticsearch-8.14.0
bin/elasticsearch
```

拿到输出结果

```
✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  U_BCBcrg6xZ9=6Xa0cRv

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  5f26e4022ca8e19d1a7152eb80e7517840a7e162a82978223d7d1725ad616d22

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTkyLjE2OC40My4xMDc6OTIwMCJdLCJmZ3IiOiI1ZjI2ZTQwMjJjYThlMTlkMWE3MTUyZWI4MGU3NTE3ODQwYTdlMTYyYTgyOTc4MjIzZDdkMTcyNWFkNjE2ZDIyIiwia2V5IjoiQ3hvei00OEIwRTIxWDVnaDREU1Q6SE1leTgyaWxUMnVjQU1QWUdpdjkydyJ9

ℹ️  Configure other nodes to join this cluster:
• On this node:
  ⁃ Create an enrollment token with `bin/elasticsearch-create-enrollment-token -s node`.
  ⁃ Uncomment the transport.host setting at the end of config/elasticsearch.yml.
  ⁃ Restart Elasticsearch.
• On other nodes:
  ⁃ Start Elasticsearch with `bin/elasticsearch --enrollment-token <token>`, using the enrollment token that you generated.
```

把elastic的密码存到shell的环境变量里面

```bash
export ELASTIC_PASSWORD="U_BCBcrg6xZ9=6Xa0cRv"
```

测试一下

```bash
curl --cacert config/certs/http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200
```

![image-20240609121805708](https://cdn.basi-a.top/images/image-20240609121805708.png)

## Kibana

启动kibana

```bash
cd $HOME/ELK/kibana-8.14.0
bin/kibana
```

打开浏览器输入es的token, 等待初始化完成

![image-20240609122222532](https://cdn.basi-a.top/images/image-20240609122222532.png)

登陆之后就可以接入数据了, 先放下

![image-20240609122441789](https://cdn.basi-a.top/images/image-20240609122441789.png)


