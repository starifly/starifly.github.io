---
title: 生产环境故障处理之nginx缓存权限问题
author: starifly
date: 2020-01-31T17:45:15+08:00
categories: [nginx]
tags: [nginx]
draft: true
slug: production-env-troubleshooting-nginx-cache-permission-problem
---

故障说明：

  官网放了一段flv的视频，之前还可以播放，今天突然发现播放不了了。程序都一样，测试环境没问题，线上却播放不了。

下面说下产生问题的原因和解决办法。

1.  nginx打开网页，点击视频播放，打不来，首先从nginx的error log下手，看下能否找出一些蛛丝马迹。
    

2015/04/09 18:33:20 \[crit\] 8063#0: \*15970093 open() "/data/nginx/proxy\_temp/7/41/0000006417" failed (13: Permission denied) while reading upstream, （nginx部分error日志）

通过查询nginx的eror日志，表面上是打不来缓存目录，没法缓存数据，google搜索了下，找到了答案。

nginx原本运行的账户是root,后来基于安全考虑，我修改成了你nginx,但是缓存目录的属主和属组还是root,所以视频的缓存数据写不到缓存目录。

2.找到了原因，下面说下解决方法：查看下，运行nginx的用户。

ps -ef | grep nginx 
root      4850     1  0 Jan22 ?        00:00:00 nginx: master process /data/nginx/sbin/nginx
nginx     8862  4850  0 Apr14 ?        00:00:34 nginx: worker process 
root     20476 20395  0 10:49 pts/1    00:00:00 grep nginx

3.可以看到运行nginx服务的用户是nginx用户，修改缓存目录的属主和属组为nginx。

chown -R nginx.nginx proxy\_temp
ls -ld proxy\_temp
drwx------ 12 nginx nginx 4096 Jan  8 18:02 proxy\_temp

或者是把proxy\_temp删除，就可以了。

## Reference

- [生产环境故障处理之nginx缓存权限问题](https://blog.51cto.com/taokey/1652911)
