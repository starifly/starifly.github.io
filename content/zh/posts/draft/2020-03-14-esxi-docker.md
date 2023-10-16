---
title: 在esxi虚拟机上运行docker容器后ping不通？
author: starifly
date: 2020-03-14T19:31:45+08:00
lastmod: 2020-03-14T19:31:45+08:00
categories: [docker]
tags: [docker,网络,linux]
draft: true
slug: esxi-docker
---

Linux 虚拟机是装在 esxi 主机上的，当安装好 docker ，并运行容器后发现容器和宿主机之间网络不通，折腾了几天，换了几台 esxi 主机实验，终于把问题解决了。

关键点是 esxi 主机必须要开启混杂模式:

1. 首先通过浏览器输入 IP 地址登录主机，找到端口组，需要将端口组属性中的VLAN ID设置成全部(4095)，并确认下安全中的混杂模式

2. vswitch->编辑->安全->设置混杂模式为“接受”
