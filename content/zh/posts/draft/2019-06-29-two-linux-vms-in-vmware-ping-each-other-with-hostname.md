---
title: vmware中两台linux虚拟机使用主机名互ping
author: starifly
date: 2019-06-29T18:18:44+08:00
categories: [网络]
tags: [linux,vmwarea,虚拟机]
draft: true
slug: two-linux-vms-in-vmware-ping-each-other-with-hostname
---

现有两台虚拟机，一台是 Ubuntu 18.04，另外一台是 centos 7，要实现两台虚拟机通过主机名互 ping，最简便的办法是分别在两台虚拟机上通过 hosts 文件设置主机名和 ip 对应关系，但是如果机器一多，就需要每次都在每台机器上添加，而通过另外一种方法，也就是指定域名服务器来一次性实现主机名互通。

当虚拟机是通过 NAT 方式与主机连接，它自带了一个域名服务器（一般是 x.x.x.2），此时我们可以通过这个域名服务器来实现虚拟机之间的主机名互通。

首先设置 Ubuntu 虚拟机，打开终端，输入`sudo vim /etc/systemd/resolved.conf`，按如下修改：

```conf
DNS=192.168.50.2
Domains=localdomain
```

如果桌面网络图标显示问号，可以输入`sudo service network-manager start`解决。

*注意在 Ubuntu 上不要配置 /etc/resolve.conf 文件，因为这个文件当系统重启或网络修改时会动态地变动*

其次设置 centos 虚拟机，打开终端，输入`sudo vim /etc/resolv.conf`，按如下修改：

```conf
search localdomain
nameserver 192.168.50.2
```

至此，VMware 主机间就能通过主机名互 ping。

## Reference

- [Ubuntu18.04的网络配置](https://www.cnblogs.com/opsprobe/p/9979234.html)
- [VMware安装Centos7超详细过程（图文）](https://blog.csdn.net/babyxue/article/details/80970526)


