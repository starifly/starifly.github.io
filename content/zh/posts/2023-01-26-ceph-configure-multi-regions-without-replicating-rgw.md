---
title: ceph在不复制rgw的情况下配置多个区域
author: starifly
date: 2023-01-26T18:56:35+08:00
lastmod: 2023-01-26T18:56:35+08:00
categories: [ceph]
tags: [ceph,rgw]
draft: false
slug: ceph-configure-multi-regions-without-replicating-rgw
---

ceph 在同一个集群配置多个zone，但不同 zone 之间的 rgw 不复制，这种情形应该可以适应多租户环境，因为希望每个租户之间的数据是相互独立的，具体配置可以参考 <https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/3/html-single/object_gateway_guide_for_red_hat_enterprise_linux/index#configuring-multiple-zones-without-replication-rgw>
