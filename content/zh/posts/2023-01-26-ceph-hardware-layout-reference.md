---
title: ceph硬件布置参考
author: starifly
date: 2023-01-26T19:06:05+08:00
lastmod: 2023-01-26T19:06:05+08:00
categories: [ceph]
tags: [ceph]
draft: false
slug: ceph-hardware-layout-reference
---

1、网络拓扑参考

![](/images/ceph-01.jpg)

2、设备位置图参考

![](/images/ceph-02.jpg)

3、服务器配置信息及运行服务统计

一般来说，内存越多越好。

对于一个中等规模的集群，监视/管理器节点可以使用64GB；对于具有数百个osd的大型集群，128GB是一个合理的目标。

OSD节点一般情况下每1T硬盘对应1G内存，详见egon整理的项目硬件参数附件，或者参考下述配置也可以

最低硬件建议，详解见<https://docs.ceph.com/en/latest/start/hardware-recommendations/>

![](/images/ceph-03.jpg)

![](/images/ceph-04.jpg)

![](/images/ceph-05.jpg)

![](/images/ceph-06.jpg)

## Reference

- [附录2、ceph安装配置介绍与主机优化](https://zhuanlan.zhihu.com/p/387112707)
