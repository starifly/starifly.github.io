---
title: vsphere web client克隆centos虚拟机后修改网络
author: starifly
date: 2020-03-06T21:44:49+08:00
lastmod: 2020-03-06T21:44:49+08:00
categories: [linux]
tags: [linux,centos,网络,vsphere]
draft: true
slug: vsphere-web-client-clones-centos-virtual-machine-to-modify-network
---

在 vsphere web client 上我们可以直接对虚拟机进行克隆，以便省掉重新安装虚拟机的麻烦。但是当克隆完虚拟机并启动新的虚拟机后，我们发现系统会自动给新虚拟机分配一个 IP 地址，而且网卡名称也变了，这时我们就得重新配置一下网络。

首先执行 `nmcli con` 命令查看新网卡的 uuid 和 mac 地址，然后打开 /etc/udev/rules.d/70-persistent-net.rules（有可能是/etc/udev/rules.d/90-persistent-net.rules），修改其中的 mac 地址和网卡名称。接下来执行命令 `mv /etc/sysconfig/network-scripts/ifcfg-old /etc/sysconfig/network-scripts/ifcfg-new` 改变网卡文件名称为新的名称，并打开 /etc/sysconfig/network-scripts/ifcfg-new 文件修改 HWADDR、UUID、DEVICE 和 NAME，最后执行 `service network restart` 重启网络。
