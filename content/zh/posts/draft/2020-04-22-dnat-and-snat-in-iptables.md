---
title: iptables中DNAT和SNAT转发的配置方法
author: starifly
date: 2020-04-22T22:46:55+08:00
lastmod: 2020-04-22T22:46:55+08:00
categories: [网络,iptables]
tags: [网络,iptables,防火墙,nat,snat,dnat]
draft: true
slug: dnat-and-snat-in-iptables
---

首先将防火墙改为转发模式：`echo 1 > /proc/sys/net/ipv4/ip_forward`，永久开启：

```
[root@firewall network-scripts]# vim /etc/sysctl.conf 
..省略内容，末行添加下段内容
net.ipv4.ip_forward=1	'//开启路由转发功能'
[root@firewall network-scripts]# sysctl -p	'//刷新sysctl.conf配置'
net.ipv4.ip_forward = 1
```

然后配置 DNAT 和 SNAT

```shell
iptables -t nat -A PREROUTING -d 202.202.202.1 -p tcp --dport 80 -j DNAT --to-destination 192.168.0.101:80  
iptables -t nat -A POSTROUTING -d 192.168.0.101 -p tcp --dport 80 -j SNAT --to 192.168.0.1
```

一般是在 prerouting 链做 DNAT，在 postrouting 链做 SNAT，但是也可以在 output 链中做 DNAT，如：

```shell
iptables -t nat -A OUTPUT --destination 192.168.83.10 -p tcp --dport 22 -j DNAT --to-destination 192.168.83.12
```

这样我们 `ssh 192.168.83.10`，会登录到 192.168.83.12 这台机器上。

## Reference

- [iptables中DNAT和SNAT转发的配置方法](https://www.cnblogs.com/liawne/p/9524519.html)
- [iptables 的OUTPUT 链中做DNAT](http://blog.chinaunix.net/uid-27571599-id-3365534.html)
