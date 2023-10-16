---
title: zabbix监控内存使用百分比
author: starifly
date: 2020-05-18T21:28:14+08:00
lastmod: 2020-05-18T21:28:14+08:00
categories: [zabbix]
tags: [zabbix]
draft: true
slug: zabbix-monitors-the-percentage-of-mem-used
---

zabbix 4.4 中没有自带的监控 linux 系统内存使用百分比的监控项，所以需要我们自己新建监控项，如下：

监控项名称：Memory used percent

类型： 可计算的

键值： vm.memory.size[usedpercent]

公式： (1-last("vm.memory.size[available]"/last("vm.memory.size[total]"))\*100)

信息类型：浮点数

单位：%
