---
title: 配置VLAN交换机支持IPTV和网络单线复用
author: starifly
date: 2023-01-26T16:45:14+08:00
lastmod: 2023-01-26T16:45:14+08:00
categories: [网络,交换机]
tags: [网络,交换机,vlan,iptv]
draft: true
slug: configure-vlan-switch-to-support-iptv-and-network-single-line-multiplex
---

VLAN交换机的划分网段也很简单，下面以水星（MERCURY）SG105 Pro 5口交换机为类，分别设置两台VLAN交换机的网段。水星SG105 Pro 交换机设置起来也比较简单，首先要进入交换机，水星SG105 Pro交换机的网址为：192.168.0.1。如果电脑有线连接交换机，需把电脑的有线网口设为和交换机相同的网段：如下图，把电脑的以太网口设为：192.168.0.10。

上图的两台VLAN交换机设置一样的，因为两台交换机所用的口是一样的，1口用于连接电视TV，2口用于连接网络NET，5口是电视TV和网络NET共用。

单独的VLAN选择Untagged端口,共用的VLAN选择Tagged端。下面电脑连接交换机，在浏览器输入;192.168.0.1，就可以进入水星SG105 Pro交换机设置页面。用户名和密码都是:admin

水星SG105 Pro交换机主要设置两项: 802.1Q VLAN和802.1Q PVID设置。都在VLAN菜单里设置。

一，先设置802.1Q VLAN：

1.第一步启用802.1Q VLAN,点击启用，应用。

2.设置电视TV，VLAN号为10，VLAN描述为TV,Untagged端口为1，Tagged端口为5，点击下面的“添加/编辑”。

3.设置网络NET，VLAN号为11，VLAN描述为NET，Untagged端口为2，Tagged端口为5，点击下面的“添加/编辑”。

## Reference

- [VLAN交换机设置教程一：单线复用，Mesh组网单线复用](https://zhuanlan.zhihu.com/p/432478315)
- [家庭Mesh组网有线回程布线方案](https://zhuanlan.zhihu.com/p/367031559)
- [用一根网线同时使用路由器和安装IPTV机顶盒，路由器IPTV功能的使用方法](https://www.bilibili.com/video/av296400437/)
