---
title: Ceph 硬件选型、crush规则及region
author: starifly
date: 2021-10-06T17:14:18+08:00
lastmod: 2021-10-06T21:00:00
categories: [ceph]
tags: [ceph,crush]
draft: false
slug: ceph-hardware-planning-crush-rules-and-region
---

## Reference

- [附录2、ceph安装配置介绍与主机优化](https://zhuanlan.zhihu.com/p/387112707)
- [[ ceph  ] CEPH 部署完整版（CentOS 7 + luminous）](https://www.cnblogs.com/hukey/p/11975109.html)
- [03 分布式存储ceph之crush规则配置](https://zhuanlan.zhihu.com/p/386560600)
- [动态设置bucket_index_max_shards参数](http://www.strugglesquirrel.com/2018/06/27/%E5%8A%A8%E6%80%81%E8%AE%BE%E7%BD%AEbucket-index-max-shards%E5%8F%82%E6%95%B0/)
- [实弹军演-基于Ceph对象存储的实战兵法](https://cloud.tencent.com/developer/article/1032859?from=article.detail.1630684)
- [从传统运维到云运维演进历程之软件定义存储（一）](https://cloud.tencent.com/developer/article/1411454?from=article.detail.1411467)
- [从传统运维到云运维演进历程之软件定义存储（三）上](https://cloud.tencent.com/developer/article/1411459?from=article.detail.1411457)
- [从传统运维到云运维演进历程之软件定义存储（三）下 ](https://cloud.tencent.com/developer/article/1411461?from=article.detail.1411457)
- [从传统运维到云运维演进历程之软件定义存储（五）上 ](https://cloud.tencent.com/developer/article/1411463?from=article.detail.1411465)
- [从传统运维到云运维演进历程之软件定义存储（五）中 ](https://cloud.tencent.com/developer/article/1411465)
- [从传统运维到云运维演进历程之软件定义存储（五）下](https://cloud.tencent.com/developer/article/1411467?from=article.detail.1411465)
- [Ceph-RGW多活机制](https://www.sohu.com/a/122822222_468741)
- [BlueStore-先进的用户态文件系统《一》](https://zhuanlan.zhihu.com/p/45084771)
- [Ceph之对象存储网关RADOS Gateway(RGW)](http://t.zoukankan.com/dengchj-p-9444437.html)
