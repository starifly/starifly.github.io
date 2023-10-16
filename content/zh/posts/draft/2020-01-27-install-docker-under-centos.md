---
title: centos7安装docker
author: starifly
date: 2020-01-27T14:11:59+08:00
categories: [docker]
tags: [docker,centos,linux]
draft: true
slug: install-docker-under-centos
---

**Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10**

## 卸载旧版本Docker

较旧的 Docker 版本称为 docker 或 docker-engine 。如果已安装这些程序，请卸载它们以及相关的依赖项。

```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine
```

## 安装 Docker

安装 Docker 有两种方式，一是通过 yum 安装，二是通过脚本自动安装，选择其中一种方法安装即可。

### 使用 Docker 仓库进行安装

1、确保服务器连网，配置网络Yum源，安装docker需要extra源

参考 《[centos7安装阿里云源](https://github.com/starifly/blog/blob/master/content/draft/2019-10-20-centos7-install-aliyun-source.md)》

2、安装Docker依赖

```shell
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

3、配置Docker的Yum源

```shell
$ sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

4、安装Docker CE

```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

**要安装特定版本的 Docker Engine-Community，请在存储库中列出可用版本，然后选择并安装：**

```shell
$ yum list docker-ce --showduplicates | sort -r
```

### 使用脚本安装 Docker

```shell
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```

### 启动Docker并加入开机启动

```shell
$ sudo systemctl start docker
$ sudo systenctl enabel docker
```

### 建立 docker 用户组

默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。

将当前用户加入 docker 组：

`$ sudo usermod -aG docker $USER`

退出当前终端并重新登录

查看docker安装版本信息：

`$ docker info`

## 安装 docker-compose

```shell
# curl -L https://github.com/docker/compose/releases/download/1.9.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# chmod +x /usr/local/bin/docker-compose
# exit
$ docker-compose version
```

## Reference

- [Centos7下安装Docker（详细安装教程）](https://blog.csdn.net/u014069688/article/details/100532774)
- [CentOS Docker 安装](https://www.runoob.com/docker/centos-docker-install.html)
- [Centos7安装docker](https://jingyan.baidu.com/article/6fb756ecf99ab6641858fb86.html)
- [centos7安装docker](https://www.jianshu.com/p/3b7d47a8b891)
- [CentOS7安装Docker](https://www.jianshu.com/p/3a4cd73e3272)
