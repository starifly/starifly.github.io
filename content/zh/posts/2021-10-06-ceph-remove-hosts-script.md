---
title: ceph剔除主机脚本
author: starifly
date: 2021-10-06T19:41:13+08:00
lastmod: 2021-10-06T19:41:13+08:00
description: ceph剔除主机操作步骤
categories: [ceph]
tags: [ceph]
draft: false
slug: ceph-remove-hosts-script
---

## 操作步骤

　　1. Monitor操作，将OSD的状态设置Out

　　2. 登录对应OSD节点 ，停止OSD守护进程

　　3. Crushmap中删除OSD

　　4. 删除OSD的认证信息

　　5. 登录对应OSD节点 ，取消挂载文件系统

## 注意事项

停止OSD和删除Crushmao的信息，都将导致PG的重新收敛，所以最好每一个步骤都确保当前ceph状态是OK才进行下一步。

　　######参考脚本

```sh
#!/bin/sh
####judge state of ceph
judge() {
  while true
  do
    sleep 5
    ceph -s |grep HEALTH_OK
      if [[ $? -eq 0 ]]
      then
        break
      else
        sleep 60
      fi
  done
}
####rdel osd from host14
while true
do
  for host in `seq 31 39`
  do
    echo "now.... ,del osd.$host from host14" >>stat.txt
    ceph osd out $host
    judge
    ssh 192.168.220.14 "/etc/init.d/ceph stop osd.$host"
    judge
    ceph osd crush remove osd.$host
    judge
    ceph auth del osd.$host
    judge
    ceph osd rm osd.$host
    judge
    ssh 192.168.220.14 "umount /var/lib/ceph/osd/ceph-$host"
    judge
    echo "del osd.$host from host14 suceess!!!!" >>stat.txt
  done
  break
done

######del osd from host16
while true
do
  for host in `seq 40 45`
  do
    echo "now.... ,del osd.$host from host16" >>stat.txt
    ceph osd out $host
    judge
    ssh 192.168.220.16 "/etc/init.d/ceph stop osd.$host"
    judge
    ceph osd crush remove osd.$host
    judge
    ceph auth del osd.$host
    judge
    ceph osd rm osd.$host
    judge
    ssh 192.168.220.16 "umount /var/lib/ceph/osd/ceph-$host"
    judge
    echo "del osd.$host from host16 suceess!!!!" >>stat.txt
  done
  break
done
```

## Reference

- [Ceph剔除主机，Crushmap遗留脏数据](https://my.oschina.net/u/4383702/blog/3678436)
