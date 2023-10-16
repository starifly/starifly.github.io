---
title: centos7修改系统时间
author: starifly
date: 2019-09-22T17:12:44+08:00
categories: [linux]
tags: [linux,centos]
draft: true
slug: centos7-modify-system-time
---

```shell
#将硬件时钟调整为与本地时钟一致, 0 为设置为 UTC 时间
timedatectl set-local-rtc 1
#设置系统时区为上海
timedatectl set-timezone Asia/Shanghai
#安装ntp
yum -y install ntp
#通过阿里云时间服务器校准时间
ntpdate ntp1.aliyun.com
#查看硬件时间
hwclock --show
#同步系统时间和硬件时间
hwclock --systohc
#保存时钟
clock -w
```

## Reference

- [centos7查看修改时区](https://blog.csdn.net/superlover_/article/details/83655646)
- [CentOS7 永久修改系统时间](https://blog.csdn.net/weixin_43196422/article/details/86584564)
