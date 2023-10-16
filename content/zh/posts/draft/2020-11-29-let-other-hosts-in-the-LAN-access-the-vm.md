---
title: 如何让局域网中的其他主机访问虚拟机
author: starifly
date: 2020-11-29T15:16:59+08:00
lastmod: 2020-11-29T15:16:59+08:00
categories: [网络]
tags: [网络,虚拟机,vmware]
draft: true
slug: let-other-hosts-in-the-LAN-access-the-vm
---

现状是家里有两台笔记本电脑，旧电脑上也安装了一些虚拟机，但是使用的是nat网络，这就面临一个问题，要如何在另外一台电脑上用xshell访问旧电脑上的虚拟机，方法是进行端口映射，打开虚拟机软件菜单栏“编辑”——“虚拟网络编辑器”，选择NAT模式，点击下面的更改设置，添加对应虚拟机的ip地址、虚拟机对应端口和主机对应端口，确定即可。

做完配置，原本以为可以开心地连接上了，在新电脑上打开xshell，尝试用telnet发起连接，结果一点反应都没有，于是又确认了配置是没错的，而且在本机连映射的端口是可以通的，没办法这时只能派wireshark上场了，通过抓包分析，在新电脑上的连接请求产生了“Tcp Retransmission”，这是tcp的超时重传机制，发送方没有得到ACK回复，这时想到是不是被防火墙阻止了，果然关闭防火墙后就可以顺利连接上了，但是防火墙还是要打开的，重新开启防火墙并添加一条入站规则即可。

## Reference

- [如何让局域网中的其他主机访问虚拟机](https://www.cnblogs.com/mkl34367803/p/10095055.html)
- [tcp retransmission原因](https://blog.csdn.net/lemontree1945/article/details/88581516)
