---
keywords:
- 
title: "部署高可靠ingress"
date: 2023-11-04T09:57:51+08:00
lastmod: 2023-11-04T09:57:51+08:00
description: ""
draft: true 
author: starifly
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
categories: [k8s,ingress]
tags: [k8s,ingress]
slug: deploy-highly-reliable-ingress
img:
---

## Reference

- [Ingress企业实战：部署高可靠性Ingress篇-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2327371)
- [什么是Ingress_容器服务 Kubernetes 版 ACK-阿里云帮助中心](https://help.aliyun.com/zh/ack/ack-managed-and-ack-dedicated/user-guide/ingress-overview)
- [Nginx Ingress管理_容器服务 Kubernetes 版 ACK-阿里云帮助中心](https://help.aliyun.com/zh/ack/ack-managed-and-ack-dedicated/user-guide/nginx-ingress-management/?spm=a2c4g.11186623.0.0.64be269b7cbBBd)
- [如何部署一套高性能、高可靠的Ingress接入层_容器服务 Kubernetes 版 ACK-阿里云帮助中心](https://help.aliyun.com/zh/ack/ack-managed-and-ack-dedicated/user-guide/deploy-ingresses-in-a-high-reliability-architecture)
- [如何通过部署NginxIngressController来支撑高负载应用_容器服务 Kubernetes 版 ACK-阿里云帮助中心](https://help.aliyun.com/zh/ack/ack-managed-and-ack-dedicated/user-guide/install-the-nginx-ingress-controller-in-high-load-scenarios)
- [如何在容器服务ACK中部署多个IngressController_容器服务 Kubernetes 版 ACK-阿里云帮助中心](https://help.aliyun.com/zh/ack/ack-managed-and-ack-dedicated/user-guide/deploy-multiple-ingress-controllers-in-a-cluster)
- [Ingress企业实战：部署多个Ingress控制器-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2327643)
- [本文对比了Nginx Ingress、ALB Ingress和MSE Ingress三种产品的功能、架构和性能差异_容器服务 Kubernetes 版 ACK-阿里云帮助中心](https://help.aliyun.com/zh/ack/ack-managed-and-ack-dedicated/user-guide/comparison-among-nginx-ingresses-alb-ingresses-and-mse-ingresses-1#task-2273852)
- [NginxIngress有哪些最佳实践_容器服务 Kubernetes 版 ACK-阿里云帮助中心](https://help.aliyun.com/zh/ack/ack-managed-and-ack-dedicated/user-guide/best-practices-for-the-nginx-ingress-controller-2#task-2175597)
