---
keywords:
- 
title: "使用Service实现简单的灰度发布和蓝绿发布"
date: 2023-11-04T10:05:25+08:00
lastmod: 2023-11-04T10:05:25+08:00
description: ""
draft: true 
author: starifly
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
categories: [k8s]
tags: [k8s,service]
slug: gray-release-and-blue-green-release-by-service
img:
---

用Kubernetes原生的特性实现简单的灰度发布和蓝绿发布。

灰度发布，也叫金丝雀发布，一般是两种版本共存，让用户使用一段时间，如果没有问题，则把流量全部迁移到新版本上。

蓝绿发布，一共有两套系统：一套是正在提供服务系统，标记为“绿色”；另一套是准备发布的系统，标记为“蓝色”。两套系统都是功能完善的、正在运行的系统，只是系统版本和对外服务情况不同。

## 利用svc标签灰度发布

以Deployment为例，用户通常会为每个Deployment创建一个Service，但Kubernetes并未限制Service需与Deployment一一对应。Service通过selector匹配后端Pod，若不同Deployment的Pod被同一selector选中，即可实现一个Service对应多个版本Deployment。调整不同版本Deployment的副本数，即可调整不同版本服务的权重，实现灰度发布。示意图如下：

![](/images/k8s_deploy_01.png)

**步骤1：部署两个版本的服务**

在集群中部署两个版本的Nginx服务，通过ELB对外提供访问。

1.创建第一个版本的Deployment，本文以nginx-v1为例。YAML示例如下：

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-v1
spec:
  replicas: 2               # Deployment的副本数，即Pod的数量
  selector:                 # Label Selector（标签选择器）
    matchLabels:
      app: nginx
      version: v1
  template:
    metadata:
      labels:               # Pod的标签
        app: nginx
        version: v1
    spec:
      containers:
      - image: {your_repository}/nginx:v1 # 容器使用的镜像为：nginx:v1
        name: container-0
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
      imagePullSecrets:
      - name: default-secret
```

2.创建第二个版本的Deployment，本文以nginx-v2为例。YAML示例如下：

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-v2
spec:
  replicas: 2               # Deployment的副本数，即Pod的数量
  selector:                 # Label Selector（标签选择器）
    matchLabels:
      app: nginx
      version: v2
  template:
    metadata:
      labels:               # Pod的标签
        app: nginx
        version: v2
    spec:
      containers:
      - image: {your_repository}/nginx:v2   # 容器使用的镜像为：nginx:v2
        name: container-0
        resources:
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
      imagePullSecrets:
      - name: default-secret
```

**步骤2：实现灰度发布**

1.为部署的Deployment创建LoadBalancer类型的Service对外暴露服务，selector中不指定版本，让Service同时选中两个版本的Deployment的Pod。YAML示例如下：

```yml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubernetes.io/elb.id: 586c97da-a47c-467c-a615-bd25a20de39c    # ELB实例的ID，请替换为实际取值
  name: nginx
spec: 
  ports:
  - name: service0
    port: 80
    protocol: TCP
    targetPort: 80
  selector:             # selector中不包含version信息
    app: nginx
  type: LoadBalancer    # 类型为LoadBalancer
```

2.执行以下命令，测试访问。

`for i in {1..10}; do curl <EXTERNAL_IP>; done;`，其中，<EXTERNAL_IP>为ELB实例的IP地址。返回结果如下，一半为v1版本的响应，一半为v2版本的响应。

```sh
Nginx-v2
Nginx-v1
Nginx-v1
Nginx-v1
Nginx-v2
Nginx-v1
Nginx-v2
Nginx-v1
Nginx-v2
Nginx-v2
```

3.通过控制台或kubectl方式调整Deployment的副本数，将v1版本调至4个副本，v2版本调至1个副本。

```sh
kubectl scale deployment/nginx-v1 --replicas=4
kubectl scale deployment/nginx-v2 --replicas=1
```

4.执行以下命令，再次测试访问。

`for i in {1..10}; do curl <EXTERNAL_IP>; done;`，其中，<EXTERNAL_IP>为ELB实例的IP地址。返回结果如下，可以看到10次访问中仅2次为v2版本的响应，v1与v2版本的响应比例与其副本数比例一致，为4:1。通过控制不同版本服务的副本数就实现了灰度发布。

```sh
Nginx-v1
Nginx-v1
Nginx-v1
Nginx-v1
Nginx-v2
Nginx-v1
Nginx-v2
Nginx-v1
Nginx-v1
Nginx-v1
```

> 如果10次访问中v1和v2版本比例并非4:1，可以将访问次数调整至更大，比如20。理论上来说，次数越多，v1与v2版本的响应比例将越接近于4:1。

## 利用滚动更新实现灰度发布

除了上面通过标签的方式实现灰度发布，还可以通过滚动更新的方式实现。

打开一个标签1监测更新过程

```sh
kubectl get pods -l app=myapp -n blue-green  -w
```

打开另一个标签2执行如下操作：

```sh
kubectl set image deployment myapp-v1 myapp=myapp:v2 -n blue-green  && kubectl rollout pause deployment myapp-v1 -n blue-green
```

> 上面的命令把myapp这个容器的镜像更新到myapp:v2 更新镜像之后，创建一个新的pod就立即暂停，这就是我们说的金丝雀发布；如果暂停几个小时之后没有问题，那么取消暂停，就会依次执行后面步骤，把所有pod都升级。

解除暂停：

```sh
kubectl rollout resume deployment myapp-v1 -n blue-green 
```

## 蓝绿发布

以Deployment为例，集群中已部署两个不同版本的Deployment，其Pod拥有共同的label。但有一个label值不同，用于区分不同的版本。Service使用selector选中了其中一个版本的Deployment的Pod，此时通过修改Service的selector中决定服务版本的label的值来改变Service后端对应的Pod，即可实现让服务从一个版本直接切换到另一个版本。示意图如下：

![](/images/k8s_deploy_02.png)

1.为部署的Deployment创建LoadBalancer类型的Service对外暴露服务，指定使用v1版本的服务。YAML示例如下：

```yml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubernetes.io/elb.id: 586c97da-a47c-467c-a615-bd25a20de39c    # ELB实例的ID，请替换为实际取值
  name: nginx
spec:
  ports:
  - name: service0
    port: 80
    protocol: TCP
    targetPort: 80
  selector:             # selector中指定version为v1
    app: nginx
    version: v1
  type: LoadBalancer    # 类型为LoadBalancer 
```

2.执行以下命令，测试访问。

`for i in {1..10}; do curl <EXTERNAL_IP>; done;`，其中，<EXTERNAL_IP>为ELB实例的IP地址。返回结果如下，均为v1版本的响应。

```sh
Nginx-v1
Nginx-v1
Nginx-v1
Nginx-v1
Nginx-v1
Nginx-v1
Nginx-v1
Nginx-v1
Nginx-v1
Nginx-v1
```

3.通过控制台或kubectl方式修改Service的selector，使其选中v2版本的服务。

```sh
kubectl patch service nginx -p '{"spec":{"selector":{"version":"v2"}}}'
```

4.执行以下命令，再次测试访问。

`for i in {1..10}; do curl <EXTERNAL_IP>; done;`，其中，<EXTERNAL_IP>为ELB实例的IP地址。返回结果如下，均为v2版本的响应，成功实现了蓝绿发布。

```sh
Nginx-v2
Nginx-v2
Nginx-v2
Nginx-v2
Nginx-v2
Nginx-v2
Nginx-v2
Nginx-v2
Nginx-v2
Nginx-v2
```

## Reference

- [使用Service实现简单的灰度发布和蓝绿发布_云容器引擎 CCE_最佳实践_发布_华为云](https://support.huaweicloud.com/intl/zh-cn/bestpractice-cce/cce_bestpractice_10002.html)
- [【精选】k8s 应用更新策略：灰度发布和蓝绿发布_k8s的配置文件指定蓝色发布_笨小孩@GF 知行合一的博客-CSDN博客](https://blog.csdn.net/gaofei0428/article/details/121413988)
- [K8S的灰度发布、滚动更新、蓝绿发布_k8s灰度发布_小仓i的博客-CSDN博客](https://blog.csdn.net/qq_42494960/article/details/119385952)
