---
title: Linux中病毒后的处理过程
author: starifly
date: 2020-05-30T21:59:57+08:00
lastmod: 2020-05-30T21:59:57+08:00
categories: [linux]
tags: [linux,centos]
draft: false
slug: virus-process-in-linux-system
---

之前一台 centos 虚拟机中毒，导致CPU飙升，这里记录下处理过程。

**查找可疑进程**

执行命令`ll /proc/ID/exe`，通过进程ID确定其执行路径，先删掉可疑应用，然后再利用`chattr +ai /path`再次生成，并 kill 掉可疑进程。

**删除定时任务及文件以及开机启动文件**

进入以下目录`/etc/init.d/`、`/etc/rc.d/rc#.d/`、`/etc/cron.d`、`/usr/lib/systemd/system/`、`/etc/systemd/system/`、`/etc/profile.d`，并清理可疑文件。

确认文件`/etc/crontab`中是否有定时任务。

**删除可疑执行文件**

进入以下目录`/bin`，`/sbin`，`/usr/libexec/`，并清理可疑文件。

**修复ssh认证文件**

`~/.ssh/authorized_keys`删除或修复

**查看etc目录下文件**

查看`/etc`目录下的文件是否被篡改或增加，比如`hosts`文件。

## Reference

- [记录一次linux病毒清除过程](https://blog.csdn.net/heivy/article/details/50508354)
- [记录一次清除Linux挖矿病毒的经历(sysupdate, networkservice进程)](https://blog.csdn.net/daiyuhe/article/details/95683393)
- [11个步骤完美排查Linux机器是否已经被入侵](https://mp.weixin.qq.com/s/eOQy1_rJdwdYcBZB0Mf-yg)
