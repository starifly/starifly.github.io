---
title: ceph rgw创建新的region
author: starifly
date: 2021-10-06T19:35:17+08:00
lastmod: 2021-10-06T19:35:17+08:00
description: 本文介绍了在ceph中利用rgw创建新的region
categories: [ceph]
tags: [ceph,rgw,region]
draft: false
slug: ceph-rgw-create-a-new-region
---

概念：      
    1、zone：可用区，有一个或多个对象网关实例组成。zone不可以跨集群，配置zone不同于其他典型配置，因为不需要在ceph.conf中配置。  
    2、zonegroup：以前叫做“region”，有多个zone组成,一个zonegroup里面有一个master zone，在同一个zonegroup中的多个zone可以同步元数据和数据，提供灾难恢复能力。  
    3、realm：代表一个唯一的命名空间，有一个或多个zonegroup组成。在同一个realm中的不同zonegroup只能同步元数据。在realm中有period的概念，表示zonegroup的配置状态，修改zonegroup，必须更新period。
	
创建realm

```bash
radosgw-admin realm create --rgw-realm=my-realm --default
[root@ceph01 ~]# radosgw-admin realm list
{
"default_info": "eff9b039-8c3c-4991-87f9-e9331b2c7824",
"realms": [
    "my-realm"
    ]
}
```

创建master zonegroup,先删除默认的zonegroup

`radosgw-admin zonegroup delete --rgw-zonegroup=default`

创建一个为azonggroup的zonegroup

`radosgw-admin zonegroup create --rgw-zonegroup=azonegroup --endpoints=192.168.0.39:7480 --master --default`

创建master zone,先删除默认的zone

`adosgw-admin zone delete --rgw-zone=default`

创建一个为azone的zone

`radosgw-admin zone create --rgw-zonegroup=azonegroup --rgw-zone=azone --endpoints=192.168.0.39:7480 --default --master`

创建一个auser账户用于和其它 zone同步

`radosgw-admin user create --uid="auser" --display-name="auser" --system`

用创建system账户产生的access 和secret更新zone配置

`radosgw-admin zone modify --rgw-zone=azone --access-key=BKG10IM15N8EB0I7ZE7U --secret=Cvh60vBX5ciujqRaLw3bm6wMIGmLdlJ9FB4ukOG`

更新period
 radosgw-admin period update --commit
 
 重启rgw实例
 
## Reference

- [ceph 多区域radosgw网关配置](http://www.idcat.cn/ceph-%E5%A4%9A%E5%8C%BA%E5%9F%9Fradosgw%E7%BD%91%E5%85%B3%E9%85%8D%E7%BD%AE.html)
- [动态设置bucket_index_max_shards参数](http://www.strugglesquirrel.com/2018/06/27/%E5%8A%A8%E6%80%81%E8%AE%BE%E7%BD%AEbucket-index-max-shards%E5%8F%82%E6%95%B0/)
