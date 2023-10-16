---
title: Zabbix遇到的一些问题
author: starifly
date: 2019-06-07T22:22:39+08:00
categories: [zabbix]
tags: [zabbix]
draft: true
slug: Zabbix-problems
---

### 安装问题

1. libiksemel

百度下载然后使用`yum install`安装

2. Requires: fping

```shell
yum install -y epel-release
yum install -y iksemel fping libiksemel
```

## 访问问题

selinux要关闭 或者 防火墙

### 自定义key Permission denied

```shell
vim /etc/zabbix/zabbix_agentd.conf
```

修改配置项：AllowRoot=1

### 修改zabbix重新check的时间

点击 administration—>General—>右侧下拉条选择”other”—>Refresh unsupported items (in sec)改为60（单位为秒）—->update

## 二进制安装Mysql后zabbixi遇到的问题

[二进制安装Mysql后zabbixi遇到的问题](https://github.com/starifly/blog/blob/master/content/draft/2019-08-05-centos7.6-install-Mysql5.7.21(universal%20binary%20package).md#二进制安装mysql后zabbixi遇到的问题)
