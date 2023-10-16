---
title: centos7安装阿里云源
author: starifly
date: 2019-10-20T22:47:45+08:00
categories: [linux]
tags: [linux,centos,阿里云]
draft: true
slug: centos7-install-aliyun-source
---

删除默认源

```shell
rm -rf /etc/yum.repos.d/CentOS*
```

阿里云源

```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
sed -i 's/$releasever/7/g' /etc/yum.repos.d/CentOS-Base.repo
```

EPEL 阿里云源

```shell
curl -o /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
```

生成缓存

```shell
yum clean all
yum makecache
```

查看repolist

```shell
yum repolist
```
