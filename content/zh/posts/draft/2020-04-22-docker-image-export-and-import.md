---
title: docker镜像导出导入命令
author: starifly
date: 2020-04-22T22:28:00+08:00
lastmod: 2020-04-22T22:28:00+08:00
categories: [docker]
tags: [docker]
draft: true
slug: docker-image-export-and-import
---

导出命令

```shell
docker save -o calico_cni.tar.gz calico/cni
```

导入命令

```shell
docker load -i calico_cni.tar.gz
```
