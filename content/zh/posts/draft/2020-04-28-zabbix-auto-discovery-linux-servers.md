---
title: Zabbix自动发现Linux主机
author: starifly
date: 2020-04-28T20:44:22+08:00
lastmod: 2020-04-28T20:44:22+08:00
categories: [zabbix]
tags: [zabbix,linux]
draft: true
slug: zabbix-auto-discovery-linux-servers
---

首先配置自动发现规则，然后再配置动作，动作主要是条件的设置，如下：

标签 | 名称
---- | ----
A | 接收到的值 包含 Linux
B | 自动发现状态 等于 上
C | 服务类型 等于 Zabbix 客户端
D | 主机IP地址 等于192.168.5.2-254

条件的计算方式是 A and B and C and D

添加完条件，最后就是配置操作：添加主机、添加主机群组、链接模板等。
