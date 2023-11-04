---
keywords:
- 
title: "U盘安装RedHat"
date: 2023-11-04T10:17:22+08:00
lastmod: 2023-11-04T10:17:22+08:00
description: ""
draft: false
author: starifly
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
categories: [linux,redhat]
tags: [linux,redhat]
slug: install-redhat-via-usb-disk
img:
---

刚开始网上查教程，大部分都说使用UltraISO刻录U盘启动盘，但是坑爹的是通过这种方式刻录的启动盘在安装过程中会找不到安装介质，所以不能使用这种方式。

后面查询其它的教程，得知可以使用Rufus制作，先格式化U盘，卷标名称位数不要超过11位，具体的刻录过程这里就不说明了。

## Reference

- [U盘安装红帽 Redhat Linux 8.4 有感_redhat8.4制作u盘启动_元宇宙隐市的博客-CSDN博客](https://blog.csdn.net/subicman/article/details/124840396)
- [U盘安装 Redhat Enterprise Linux ( RHEL ) 6.10_我是超级用户的博客-CSDN博客](https://blog.csdn.net/petrosofts/article/details/115549864)
- [OS: Redhat8.4实体机的安装 - lnlidawei - 博客园](https://www.cnblogs.com/lnlidawei/p/14820158.html)
- [RHEL 8 使用总结/避坑指南（一） 安装及配置SSH和防火墙 - Chuantao](https://zangchuantao.com/tech-zh/2021/rhel-8-install-ssh-firewalld-config/)
