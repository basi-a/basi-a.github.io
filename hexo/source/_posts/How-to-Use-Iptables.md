---
title: How to Use Iptables
date: 2023-05-18 14:36:12
updated: 2023-05-18 14:36:12
tags: Linux, iptables
categories: [Linux, iptables]
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
# iptables 工作过程
关于iptables的工作流程，我觉得`Arch wiki`, 这里画的特别好, ( ^ v ^ )
![iptable-run](https://cdn.basi-a.top/images/iptables.webp)
# 表（Tables）
iptables有5张表：
1. `raw` 用于配置数据包，`raw` 中的数据包不会被系统跟踪 
2. `filter` 存放所有与防火墙相关操作的默认表
3. `nat` 用于网络地址转换
4. `mangle` 用于特定数据的修改
5. `security` 用于强制访问规则

其中`filter`和`nat`最常用。
# 链（Chains）
1. INPUT链 ：处理输入数据包
2. OUTPUT链 ：处理输出数据包
3. FORWARD链 ：处理转发数据包
4. PREROUTING链 ：用于目标地址转换（DNAT）
5. POSTOUTING链 ：用于源地址转换（SNAT）
# 写了个很适合我自己的初始化脚本
```bash
#!/bin/bash
WHITE_LIST_PATH="$(pwd)/whitelist.txt"
BLACK_LIST_PATH="$(pwd)/blacklist.txt"
CIDR_REGEX_v4="^((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\/(3[0-2]|[12]?[0-9])$"
CIDR_REGEX_v6="^([0-9a-fA-F]{1,4}(:[0-9a-fA-F]{1,4})*)?(::([0-9a-fA-F]{1,4}(:[0-9a-fA-F]{1,4})*)?)?(/[0-9]{1,3})?$"

ipv4-set(){
iptables -F
iptables -X
iptables -Z

iptables -P INPUT DROP # 配置默认的不让进
iptables -P FORWARD DROP # 默认的不允许转发
iptables -P OUTPUT ACCEPT # 默认的可以出去

# 换回接口出入随意
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
# 放行ssh
SSHPORT="$(ss -4tulnp | grep "ssh" | awk '{print $5}' | awk -F":" '{print $2}')"
iptables -A INPUT -p tcp --dport "${SSHPORT}" -j ACCEPT

# 禁ICMP
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP # 白名单不受影响

}
ipv6-set(){
ip6tables -F
ip6tables -X
ip6tables -Z

ip6tables -P INPUT DROP # 配置默认的不让进
ip6tables -P FORWARD DROP # 默认的不允许转发
ip6tables -P OUTPUT ACCEPT # 默认的可以出去

# 换回接口出入随意
ip6tables -A INPUT -i lo -j ACCEPT
ip6tables -A OUTPUT -o lo -j ACCEPT
# 放行ssh
SSHPORT="$(ss -6tulnp | grep "ssh" | awk '{print $5}' | awk -F"]:" '{print $2}')"
ip6tables -A INPUT -p tcp --dport "${SSHPORT}" -j ACCEPT

# 禁ICMP
ip6tables -A INPUT -p icmpv6 --icmpv6-type echo-request -j DROP

}

iptables-set(){
ipv4-set
ipv6-set
}
# 配置ipv6 ipv4 黑白名单
Whitelist_And_Blacklist(){
    while IFS= read -r line
    do
        if [[ "$line" =~ $CIDR_REGEX_v4 ]]; then
            iptables -A INPUT -s "$line" -j ACCEPT
        elif [[ "$line" =~ $CIDR_REGEX_v6 ]]; then
            ip6tables -A INPUT -s "$line" -j ACCEPT
        else
            echo "$line error value"
        fi
    done < "$WHITE_LIST_PATH"

    while IFS= read -r line
    do
        if [[ "$line" =~ $CIDR_REGEX_v4 ]]; then
            iptables -A INPUT -s "$line" -j DROP
        elif [[ "$line" =~ $CIDR_REGEX_v6 ]]; then
            ip6tables -A INPUT -s "$line" -j DROP
        else
            echo "$line error value"
        fi
    done < "$BLACK_LIST_PATH"
}
iptables-set
Whitelist_And_Blacklist

iptables -L -n -v
ip6tables -L -n -v
```
