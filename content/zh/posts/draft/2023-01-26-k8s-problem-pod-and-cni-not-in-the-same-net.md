---
title: k8s Pod网络和cni网络不在同一个网段问题
author: starifly
date: 2023-01-26T14:56:15+08:00
lastmod: 2023-01-26T14:56:15+08:00
categories: [k8s,网络]
tags: [k8s,网络]
draft: true
slug: k8s-problem-pod-and-cni-not-in-the-same-net
---

k8s Pod网络和cni网络不在同一个网段

## 问题描述

k8s Pod网络和cni网络不在同一个网段，pod无法访问

## 原因分析

网络使用flannel vxlan模式，如下图所示node节点上pod通信通过cnio网桥进行转发，网桥作为pod的网关设备与pod处于同一个二层网络内。

生产遇到cni和pod网络处于不同网段，原因为读取/etc/cni/net.d目录默认的配置文件

## 解决方案

1. 删除默认10-default.config配置文件

2. 卸载mynet网卡

3. 重启flannel组件

## Reference

- [k8s Pod网络和cni网络不在同一个网段](https://blog.csdn.net/weixin_37844885/article/details/125620734)
