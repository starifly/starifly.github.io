---
title: redis哨兵模式搭建
author: starifly
date: 2020-11-10T21:20:32+08:00
lastmod: 2020-11-10T21:20:32+08:00
categories: [redis]
tags: [redis]
draft: true
slug: redis-sentinel-build
---

哨兵模式是基于主从模式搭建，主要修改如下配置：

```conf
# 修改redis-sentinel配置文件：sentinel.conf
# 1. 配置端口
port 26379
# 2. 保护模式修改为否，允许远程连接
protected-mode no
# 3. 设定监控地址，为对应的主redis库的内网地址
sentinel monitor mymaster 172.16.48.129 6379 1
# 4. 设定5秒内没有响应，说明服务器挂了,需要将配置放在sentinel monitor master 127.0.0.1 6379 1下面
sentinel down-after-milliseconds mymaster 5000
# 5. 设定15秒内master没有活起来，就重新选举主
sentinel failover-timeout mymaster 15000
# 6. 表示如果master重新选出来后，其它slave节点能同时并行从新master同步缓存的台数有多少个，显然该值越大，所有slave节点完成同步切换的整体速度越快，但如果此时正好有人在访问这些slave，可能造成读取失败，影响面会更广。最保定的设置为1，只同一时间，只能有一台干这件事，这样其它slave还能继续服务，但是所有slave全部完成缓存更新同步的进程将变慢。
sentinel parallel-syncs mymaster 2
# 7. 主数据库密码,需要将配置放在sentinel monitor master 127.0.0.1 6379 1下面
sentinel auth-pass mymaster 123456789
```

> 注意要打开各个slave防火墙端口。

## Reference

- [redis(一主两从三哨兵模式搭建)记录](https://www.cnblogs.com/yueminghai/p/10756780.html)
