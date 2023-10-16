---
title: 阿里云polardb使用流程
author: starifly
date: 2023-01-26T17:05:25+08:00
lastmod: 2023-01-26T17:05:25+08:00
categories: [polardb,数据库]
tags: [polardb,数据库]
draft: true
slug: polardb-usage
---

购买PolarDB（创建新集群）到可以开始使用，请参考：https://help.aliyun.com/document_detail/139453.html。通常，从购买PolarDB（创建新集群）到可以开始使用，您需要完成如下操作。

1. 购买[按量付费集群](https://help.aliyun.com/document_detail/58769.html?spm=5176.smartservice_service_robot-chat.help.3.7ba73cdaEBvFxS#task-1580301)或[购买包年包月集群](https://help.aliyun.com/document_detail/201224.html?spm=5176.smartservice_service_robot-chat.help.4.7ba73cdaEBvFxS#task-1580301)
2. [设置白名单](https://help.aliyun.com/document_detail/68506.html?spm=5176.smartservice_service_robot-chat.help.5.7ba73cdaEBvFxS#task-1580301)
3. [创建数据库账号](https://help.aliyun.com/document_detail/68508.html?spm=5176.smartservice_service_robot-chat.help.6.7ba73cdaEBvFxS#task-1580301)
4. [申请集群地址和主地址](https://help.aliyun.com/document_detail/68510.html?spm=5176.smartservice_service_robot-chat.help.7.7ba73cdaEBvFxS#task-1580301)
5. [连接数据库集群](https://help.aliyun.com/document_detail/68511.html?spm=5176.smartservice_service_robot-chat.help.8.7ba73cdaEBvFxS#task-1580301)

- POLARDB仅支持专有网络VPC（Virtual Private Cloud）。VPC是阿里云上一种隔离的网络环境，安全性比传统的经典网络更高。
- POLARDB与其他阿里云产品通过内网互通时才能发挥POLARDB的最佳性能，因此，建议将POLARDB与云服务器ECS配合使用，且与ECS创建于同一个VPC，否则POLARDB无法发挥最佳性能。如果您ECS的网络类型为经典网络，需将ECS从经典网络迁移到VPC，具体请参见[ECS实例迁移](https://help.aliyun.com/document_detail/57954.html?source=5176.11533457&userCode=wrvvs1rm&type=copy)。
- 主地址，总是连接到主节点，支持读和写操作。
- 集群地址，集群地址通过数据库代理实现。（PolarDB的数据库代理支持读写分离功能。应用程序只需连接一个集群地址，即可连接到多个节点，写请求会自动发往主节点，读请求会自动根据各节点的负载发往主节点或只读节点。）
