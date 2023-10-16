---
title: Zabbix磁盘IO监控之自动发现
author: starifly
date: 2020-04-27T22:03:19+08:00
lastmod: 2020-04-27T22:03:19+08:00
categories: [zabbix]
tags: [zabbix,disk,monitor]
draft: true
slug: auto-discovery-of-zabbix-disk-io-monitoring
---

## iostat

Zabbix并没有提供模板来监控磁盘的IO性能，所以我们需要自己来创建一个。iostat主要用于监控系统设备的IO负载情况，iostat首次运行时显示自系统启动开始的各项统计信息，之后运行iostat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。所以在使用iostat监控系统IO负载的时候，不要直接iostat取结果，而是iostat -dxkt 1 2取结果，否则得到的数据根本不正确。

```
iostat常用参数说明
-c           #仅显示CPU统计信息.与-d选项互斥.
-d           #仅显示磁盘统计信息.与-c选项互斥.
-k           #以K为单位显示每秒的磁盘请求数,默认单位块.
-t            #在输出数据时,打印搜集数据的时间.
-V           #打印版本号和帮助信息.
-x           #输出扩展信息.

![这里写图片描述](https://img-blog.csdn.net/20180621112352631?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21vZml1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
avg-cpu段:
 %user: 在用户级别运行所使用的CPU的百分比.
 %nice: nice操作所使用的CPU的百分比.
 %sys: 在系统级别(kernel)运行所使用CPU的百分比.
 %iowait: CPU等待硬件I/O时,所占用CPU百分比.
 %idle: CPU空闲时间的百分比.
Device段:
tps: 每秒钟发送到的I/O请求数.
Blk_read /s: 每秒读取的block数.
Blk_wrtn/s: 每秒写入的block数.
Blk_read:   读入的block总数.
Blk_wrtn:  写入的block总数.

![这里写图片描述](https://img-blog.csdn.net/2018062111241551?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21vZml1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
rrqm/s：每秒读请求被合并次数
wrqm/s：每秒写请求被合并次数
r/s：每秒完成的读次数
w/s：每秒完成的写次数
rkB/s：每秒读数据量（kb）
wkB/s：每秒写数据量（kb）
avgrq-sz：平均每次IO请求的扇区大小
avgqu-sz：平均每次IO请求的队列长度（越短越好）
await：平均每次IO请求等待时间（毫秒），一般的系统IO等待时间应该低于5ms，如果大于10ms就比较大了。这个时间包括了队列时间和服务时间，也就是说，一般情况下，await大于svctm，它们的差值越小，则说明队列时间越短，反之差值越大，队列时间越长，说明系统出了问题。
r_await：读的平均耗时（毫秒）
w_await：写入平均耗时（毫秒）
svctm：平均每次IO请求处理时间（毫秒），如果svctm的值与await很接近，表示几乎没有I/O等待，磁盘性能很好，如果await的值远高于svctm的值，则表示I/O队列等待太长，系统上运行的应用程序将变慢。
%util：IO队列非空比例，该参数暗示了设备的繁忙程度。一般地，如果该参数是100%表示设备已经接近满负荷运行了
```

## 在被监控端上编写自动发现和监控磁盘IO脚本

```
脚本目录/etc/zabbix/zabbix_agentd.d/scripts

1、磁盘发现脚本
disk_discovery.sh 
#!/bin/bash
diskarray=(`cat /proc/diskstats |grep -E "\bsd[a-z]\b|\bxvd[a-z]\b|\bvd[a-z]\b"|awk '{print $3}'|sort|uniq  2>/dev/null`)
length=${#diskarray[@]}
printf "{\n"
printf  '\t'"\"data\":["
for ((i=0;i<$length;i++))
do
        printf '\n\t\t{'
        printf "\"{#DISK_NAME}\":\"${diskarray[$i]}\"}"
        if [ $i -lt $[$length-1] ];then
                printf ','
        fi
done
printf  "\n\t]\n"
printf "}\n"

2、磁盘io监控脚本
 disk_status.sh 
#/bin/sh
Device=$1
DISK=$2
case $DISK in
         rrqm)
            iostat -dxkt 1 2|grep "\b$Device\b"|tail -1|awk '{print $2}'
            ;;
         wrqm)
            iostat -dxkt 1 2|grep "\b$Device\b"|tail -1|awk '{print $3}'
            ;;
          rps)
            iostat -dxkt 1 2|grep "\b$Device\b"|tail -1|awk '{print $4}'
            ;;
          wps)
            iostat -dxkt 1 2|grep "\b$Device\b" |tail -1|awk '{print $5}'
            ;;
        rKBps)
            iostat -dxkt 1 2|grep "\b$Device\b" |tail -1|awk '{print $6}'
            ;;
        wKBps)
            iostat -dxkt 1 2|grep "\b$Device\b" |tail -1|awk '{print $7}'
            ;;
        avgrq-sz)
            iostat -dxkt 1 2|grep "\b$Device\b" |tail -1|awk '{print $8}'
            ;;
        avgqu-sz)
            iostat -dxkt 1 2|grep "\b$Device\b" |tail -1|awk '{print $9}'
            ;;
        await)
            iostat -dxkt 1 2|grep "\b$Device\b" |tail -1|awk '{print $10}'
            ;;
        svctm)
            iostat -dxkt 1 2|grep "\b$Device\b" |tail -1|awk '{print $11}'
            ;;
         util)
            iostat -dxkt |grep "\b$Device\b" |tail -1|awk '{print $12}'
            ;;
esac

3、zabbix配置文件
UserParameter=disk.discovery[*],/etc/zabbix/zabbix_agentd.d/scripts/disk_discovery.sh
UserParameter=disk.status[*],/etc/zabbix/zabbix_agentd.d/scripts/disk_status.sh $1 $2
4、重启客户端服务
5、在服务端测试
zabbix_get -s 192.168.1.211 -k 'disk.discovery[*]
```

## zabbix控制台操作

### 配置-模板-template os linux-自动发现规则 -创建发现规则

### 监控项原型（键值[]中的数值必须大写，否则会报错）

### 自动发现清单中新建触发器：io.util.avg(1m)>80

### 图形原型（名称后边要带哪个磁盘的动态名称，否则会报错如下）

这里由于时间关系，省略了具体的模板和监控项添加过程。

## Reference

- [Zabbix磁盘IO监控之自动发现](https://blog.csdn.net/mofiu/article/details/80758358)
- [zabbix自动发现规则之磁盘IO监控](https://blog.51cto.com/jiay1/2064696)

