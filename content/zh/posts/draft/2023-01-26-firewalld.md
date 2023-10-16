---
title: firewalld
author: starifly
date: 2023-01-26T17:44:00+08:00
lastmod: 2023-01-26T17:44:00+08:00
categories: [网络]
tags: [网络,防火墙,firewalld]
draft: true
slug: firewalld
---

删掉旧的规则
vim /etc/firewalld/zones/public.xml
删除端口：firewall-cmd --permanent --zone=public --remove-port=8080/tcp
删除IP+端口：firewall-cmd --permanent --zone=public --remove-rich-rule="rule family="ipv4" source address="10.0.5.0/24" port protocol="tcp" port="10050" accept"

添加service
```
vim /etc/firewalld/services/zookeeper.xml

<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>zookeeper</short>
  <description>Zookeeper.</description>
  <port protocol="tcp" port="2181"/>
  <port protocol="tcp" port="2888-3888"/>
</service>
```
firewall-cmd --zone=public --add-service=zookeeper --permanent
firewall-cmd --reload
查询默认的zone
firewall-cmd --get-default-zone
查看指定zone下的service
firewall-cmd --zone=public --list-service

禁止访问外网

```
# 首先必须放行进入已连接的，这里暂只针对ipv4
firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 2 -m state --state RELATED,ESTABLISHED -j ACCEPT
firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 3 -j REJECT
# 放行内网，比如dns服务器、网关及其它需主动访问的机器
firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 2 -d 192.168.4.13 -j ACCEPT
```

## Reference

- [CENTOS 7 FIREWALLD详解，添加删除策略 ](https://www.cnblogs.com/faithH/p/11811286.html)
- [firewalld关于service的操作](firewalld关于service的操作)
- [firewalld（zone与service）](https://blog.51cto.com/shuzonglu/2065615)
- [centos7安装部署kafka_2.13-2.4.1集群 ](https://www.cnblogs.com/zhangmingcheng/p/13032460.html)
