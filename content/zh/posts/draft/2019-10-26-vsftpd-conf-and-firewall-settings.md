---
title: vsftpd配置及防火墙设置
author: starifly
date: 2019-10-26T21:12:20+08:00
categories: [vsftpd]
tags: [vsftpd,iptables,i网络,防火墙]
draft: true
slug: vsftpd-conf-and-firewall-settings
---

## 主动模式配置方法

#主动式连接使用的数据通道  
connect_from_port_20=YES  
#支持数据流的被动式连接模式  
pasv_enable=NO

## 被动模式配置方法

#主动式连接使用的数据通道  
connect_from_port_20=NO  
#(默认为YES)设置是否允许被动模式   
pasv_enable=YES  
pasv_min_port=9000  #设置被动模式最小端口  
pasv_max_port=9010  #设置被动模式最大端口

## 防火墙设置

建议 ftp 服务端配置成被动模式，我们再设置防火墙规则：

```shell
iptables -A INPUT -p tcp --dport 21 -j ACCEPT
iptables -A INPUT -p tcp --dport 9000:9010 -j ACCEPT
```

> 另外还要让kenel加载FTP模块

## Reference

- [Linux安装vsftpd及配置详解](https://blog.51cto.com/meiling/2071122)
- [更改vsftpd默认目录](https://blog.51cto.com/meiling/2068013)
- [vsftp配置主动模式和被动模式](https://blog.csdn.net/cy104204/article/details/24490729)
