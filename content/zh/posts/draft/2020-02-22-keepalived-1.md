---
title: Keepalived 1
author: starifly
date: 2020-02-22T18:15:39+08:00
lastmod: 2020-02-22T18:15:39+08:00
categories: [学习笔记]
tags: [keepalived,马哥全新第30期Linux运维教程]
draft: true
slug: keepalived-1
---

1. keepalived 配置前提：

- 各节点时间必须同步
- 确保防火墙不会成为阻碍
- 确保个节点用于集群服务的接口支持 MULTICAST 通信

2. vrrp 同步组

    如果某一个应用对外（客户端）使用一个地址，对内（服务器）也使用一个地址，以代理服务器为例，接收客户端请求使用一个 IP 地址，转发内网主机使用一个 IP 地址，如果面向服务端的地址做了高可用 VIP1，对内同样做了高可用 VIP2，这时 VIP1 在哪个节点上，VIP2 就应该在哪个节点上，VRRP 同步组就实现了这样的功能，把两个 VIP 定义成一个同步组。

3. 为了充分利用主机资源，keepalived 也可以做主主，主主之上再做 DNS 主从。

# Reference

《马哥全新第30期Linux运维教程》
