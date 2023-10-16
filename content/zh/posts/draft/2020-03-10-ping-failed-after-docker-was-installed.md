---
title: 安装 Docker 后 ping 不通
author: starifly
date: 2020-03-10T22:53:20+08:00
lastmod: 2020-03-10T22:53:20+08:00
categories: [docker]
tags: [docker,网络]
draft: true
slug: ping-failed-after-docker-was-installed
---

今天准备把服务搬上 Docker，但是遇到一个奇怪得问题：容器运行后，在主机上 ping 不通容器，进入容器也 ping 不通主机，于是上网一番搜索，发现了问题所在，有可能是 Docker 和宿主机的网段冲突了，改了网段就好了。

Docker 容器网络默认使用的是 bridge 桥接模式，一般容器会使用 daemon.json 中定义的虚拟网桥来与宿主机进行通信。

下面是在 Linux 上修改 Docker 默认网段的方法。

1. 删除原有配置

```shell
sudo service docker stop
sudo ip link set dev docker0 down
sudo brctl delbr docker0
sudo iptables -t nat -F POSTROUTING
```

2. 创建新的网桥

```shell
sudo brctl addbr docker0
sudo ip addr add 172.17.10.1/24 dev docker0
sudo ip link set dev docker0 up
```

3. 配置 Docker 的文件

```shell
vi /etc/docker/daemon.json
-bash-4.2$ cat /etc/docker/daemon.json
{
	"bip":"172.17.10.1/24"
}
# 注意就是将 bip 的值改成新设置的网段
```

最后重启 Docker，问题解决。

# Reference

- [安装 Docker 后服务器 ping 不通了？](https://juejin.im/post/5dcf5f016fb9a0204359be94)
