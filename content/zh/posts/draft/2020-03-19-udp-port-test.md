---
title: UDP 端口测试方法
author: starifly
date: 2020-03-19T21:14:46+08:00
lastmod: 2020-03-19T21:14:46+08:00
categories: [网络]
tags: [网络,udp,端口]
draft: true
slug: udp-port-test
---

方案一：

**软电话**

1. 在需要开通 udp 端口的服务器上执行 `nc -uvlp udpport` 监听端口，需要注意的是如果端口此前有服务在使用，先要停掉服务在开启监听。

2. 然后再外部用软电话去注册上面监听中的端口，如果端口是通的，此时就会收到注册消息。

方案二：

**socat**

1. 在需要开通 udp 端口的服务器上执行 `nc -uvlp udpport` 监听端口，需要注意的是如果端口此前有服务在使用，先要停掉服务在开启监听。

2. `echo “hello” | socat - udp4-datagram:10.32.2.231:5080`，或者 `echo "test" | socat - udp-connect:127.0.0.1:23456`

## Reference

- [linux 网络检测常用命令(tcp/udp 端口检测)](https://blog.csdn.net/PKU1254/article/details/81137281)
- [Socat 入门教程](https://www.hi-linux.com/posts/61543.html)

