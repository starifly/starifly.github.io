---
title: cdn加速oss资源
author: starifly
date: 2023-01-26T17:09:26+08:00
lastmod: 2023-01-26T17:09:26+08:00
categories: [cdn,oss]
tags: [cdn,oss]
draft: true
slug: cdn-accelerates-oss
---

对象存储OSS与阿里云CDN服务结合，可将OSS内的文件缓存到CDN的边缘节点。当大量终端用户重复访问同一文件时，可以直接从边缘节点获取已缓存的数据，提高访问的响应速度。

当客户分布全球时，为便于客户端就近取得资源，应该在国内国外同时建有存储桶，同时两者之间利用跨区域复制技术实现双向同步。

## 准备工作

分别在华南和德国区建一个oss存储桶，配置两桶之间双向复制，同时开启全球加速功能，回源或者上传的时候利用全球加速域名。

## 添加加速域名

登录oss管理控制台，选择传输管理 > 域名管理，单击绑定域名，在绑定域名面板的域名文本框填写您的域名。

![](/images/oss-cdn-01.png)

## 配置CDN

点击oss域名管理下相应域名“未配置”按钮，会跳转到CDN配置界面。

步骤一：配置基础信息和业务信息

加速区域根据需要配置，这里选择全球。

![](/images/oss-cdn-02.png)

步骤二：配置源站

点击新增源站信息，oss域名要填写全球加速域名，以此类推添加两个源站，一个为华南，一个为德国，权重要设置一样一遍回源的时候根据客户地理位置自动回源至就近的oss桶。

![](/images/oss-cdn-03.png)

步骤二：CNAME配置

给加速域名添加CNAME，CNAME值为CDN地址。

## 开启CDN缓存自动刷新

在域名管理页签，打开目标域名右侧的CDN缓存自动刷新开关，这样当oss上的文件更新时，会自动缓存更新至cdn节点。

![](/images/oss-cdn-04.png)

## CDN优化配置

**回源配置**

回源HOST这里不用配置

**缓存配置**

缓存过期时间设置为1年，也可以根据需要调整过期时长。

![](/images/oss-cdn-05.png)

**跨域配置**

![](/images/oss-cdn-06.png)

**https配置**

![](/images/oss-cdn-07.png)

**访问配置**

开启Referer防盗链或其它安全配置

![](/images/oss-cdn-08.png)

**性能优化**

开启智能压缩和参数保留

![](/images/oss-cdn-09.png)

## oss配置

**防盗链**

![](/images/oss-cdn-10.png)

**跨域设置**

跨域设置，不然回源的时候会有问题

![](/images/oss-cdn-12.png)

## Reference

- [CDN加速OSS资源（通过OSS控制台实现）](https://help.aliyun.com/document_detail/123227.html?spm=a2c4g.11186623.0.0.5c365b29UcyrgD)
- [CDN加速OSS资源（通过CDN控制台实现）](https://help.aliyun.com/document_detail/123226.html?spm=a2c4g.11186623.0.0.5c365b29UcyrgD)
- [【百问百答】《CDN排坑指南》](https://developer.aliyun.com/ask/315119?spm=a2c6h.13066354.0.0.2ab8286521GDlG)
