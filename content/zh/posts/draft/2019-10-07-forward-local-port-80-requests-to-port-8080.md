---
title: 利用iptables将本地80端口的请求转发到8080端口
author: starifly
date: 2019-10-07T21:11:03+08:00
categories: [网络,iptables]
tags: [网络,防火墙,iptables]
draft: true
slug: forward-local-port-80-requests-to-port-8080
---

当面临 80 端口不能用的时候或者外网没有开放相应服务的端口假设为8080，除了用 nginx 做转发外，我们还可以通过 iptables 将本地 80 的请求转发到 8080 端口。

如下：

```shell
iptables -t nat -A PREROUTING -d 192.168.1.3 -p tcp --dport 80 -j DNAT --to 192.168.1.3:8080
```

或者

```shell
iptables -t nat -A PREROUTING -i eth0 -d 192.168.1.3 -p tcp -m tcp --dport 80 -j REDIRECT--to-ports 8080
```
