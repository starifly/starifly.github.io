---
title: k8s笔记与面试题
author: starifly
date: 2023-01-28T16:32:51+08:00
lastmod: 2023-02-10
categories: [k8s]
tags: [k8s,面试]
draft: true
slug: k8s-notes-and-interview-questions
---

## 笔记

1. docker可以利用--rm bash调试容器

2. docker默认的网络模式为brige（会虚拟一个docker0网卡作为网桥），但是容器和外界通信依然要做snat和dnat转换

3. k8s查看资源命令： kubectl api-resources（其中也可以看到apiversion）

4. k8s删除yaml的时候如果等待过久，可以新启一个窗口直接删除相关的容器。

5. k8s中schedule调度器和kubelet都采用list-watch机制去发现有没有新的任务。

6. k8s requests和limits区别
requests作用于schedule阶段，容器使用的最小资源需求
limits是容器能使用资源的最大值，当pod内存超过limit时，会被oom

7. k8s中为什么要统一管理环境变量
- 环境变量中有很多敏感信息，直接暴露在yaml文件中存在安全性问题
- 团队内部一般存在多个项目，这些项目直接存在配置相同环境变量的情况，因此可以统一维护管理
- 存在多个环境，如果不统一管理，由于配置均不同，每套环境部署的时候都要修改yaml，带来额外的开销

在部署不同的环境时，pod的yaml无须再变化，只需要在每套环境中维护一套ConfigMap和Secret即可。但是注意configmap和secret不能跨namespace使用，且更新后，pod内的env不会自动更新，重建后方可更新。

8. configmap的一种创建方式：

```
$ cat configmap.txt
MYSQL_HOST=172.21.51.68
MYSQL_PORT=3306
$ kubectl create configmap myblog --from-env-file=configmap.txt -n demo
```

或者更简化的方式：

```
kubectl create configmap my-config --from-literal=key1=config1 -n demo
```

9. secret的一种创建方式：

```
$ cat secret.txt
MYSQL_USER=root
MYSQL_PASSWD=123456
$ kubectl create secret generic my-secret --from-env-file=secret.txt -n demo
```
generic对应Opaque类型即base64编码格式的secret

10. 如何编写资源yaml
- 拿来主义，从机器中已有的资源中拿，$ kubectl -n kube-system get po,deployment,ds
- 学会在官网查找， https://kubernetes.io/docs/home/
- 从kubernetes-api文档中查找， https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/#pod-v1-core
- kubectl explain 查看具体字段含义

11. k8s服务更新方式：

- 修改yaml文件，使用`kubectl apply -f deploy-myblog.yaml`来应用更新
- `kubectl -n demo edit deploy myblog`在线更新
- `kubectl set image deploy myblog myblog=172.21.32.6:5000/myblog:v2 --record`

12. k8s更新策略：

- maxSurge：最大激增数, 指更新过程中, 最多可以比replicas预先设定值多出的pod数量, 可以为固定值或百分比,默认为desired Pods数的25%。计算时向上取整(比如3.4，取4)，更新过程中最多会有replicas + maxSurge个pod
- maxUnavailable： 指更新过程中, 最多有几个pod处于无法服务状态 , 可以为固定值或百分比，默认为desired Pods数的25%。计算时向下取整(比如3.6，取3)

13. 跨命名空间访问svc
跨命名空间访问service需使用 服务名.命名空间 的形式访问。

14. pod的定义中有个command和args参数，这两个参数不会和docker镜像的entrypointc冲突吗？

- 如果command和args均没有写，那么使用Dockerfile的配置；
- 如果command写了但args没写，那么Dockerfile默认的配置会被忽略，执行指定的command；
- 如果command没写但args写了，那么Dockerfile中的ENTRYPOINT的会被执行，使用当前args的参数；
- 如果command和args都写了，那么Dockerfile会被忽略，执行输入的command和args。

15. k8s认证支持的方法：

- basic auth 即用户名密码
- client证书验证
- jwt token（用于serviceaccount）

16. k8s监控指标

- 内部系统组件的状态：比如kube-apiserver、kube-schedule、kube-controller-manager、kubedns/coredns等组件的详细运行状态
- kubernets节点的监控：比如节点的cpu、load、disk、memory等指标
- 业务容器指标的监控（容器CPU、内存、磁盘等）
- 编排级的metrics：比如Deployment的状态、资源请求、调度和API延迟等数据指标

17. serviceaccount会把证书和token挂载到pod内部 

pod => serviceaccount => secret(token)，具体挂载地址为/var/run/secrets/kubernets.io/serviceaccount/

## 面试题

1、kubelet的功能、作用是什么？（重点，经常会问）

答：kubelet部署在每个node节点上的，它主要有2个功能：
1）节点管理。kubelet启动时会向api-server进行注册，然后会定时的向api-server汇报本节点信息状态，资源使用状态等，这样master就能够知道node节点的资源剩余，节点是否失联等等相关的信息了。master知道了整个集群所有节点的资源情况，这对于 pod 的调度和正常运行至关重要。
2）pod管理。kubelet负责维护node节点上pod的生命周期，当kubelet监听到master的下发到自己节点的任务时，比如要创建、更新、删除一个pod，kubelet 就会通过CRI（容器运行时接口）插件来调用不同的容器运行时来创建、更新、删除容器；常见的容器运行时有docker、containerd、rkt等等这些容器运行时，我们最熟悉的就是docker了，但在新版本的k8s已经弃用docker了，k8s1.24版本中已经使用containerd作为容器运行时了。
3）容器健康检查。pod中可以定义启动探针、存活探针、就绪探针等3种，我们最常用的就是存活探针、就绪探针，kubelet 会定期调用容器中的探针来检测容器是否存活，是否就绪，如果是存活探针，则会根据探测结果对检查失败的容器进行相应的重启策略；
4）Metrics Server资源监控。在node节点上部署Metrics Server用于监控node节点、pod的CPU、内存、文件系统、网络使用等资源使用情况，而kubelet则通过Metrics Server获取所在节点及容器的上的数据。

2、pod的存活探针有哪几种？（必须记住3重探测方式，重点，经常问）

kubernetes可以通过存活探针检查容器是否还在运行，可以为pod中的每个容器单独定义存活探针，kubernetes将定期执行探针，如果探测失败，将杀死容器，并根据restartPolicy策略来决定是否重启容器，kubernetes提供了3种探测容器的存活探针，如下：
1）httpGet：通过容器的IP、端口、路径发送http 请求，返回200-400范围内的状态码表示成功。
2）exec：在容器内执行shell命令，根据命令退出状态码是否为0进行判断，0表示健康，非0表示不健康。
3）TCPSocket：与容器的IP、端口建立TCP Socket链接，能建立则说明探测成功，不能建立则说明探测失败。

3、就绪探针与存活探针区别是什么？

两者作用不一样，存活探针是将检查失败的容器杀死，创建新的启动容器来保持pod正常工作；
就绪探针是，当就绪探针检查失败，并不重启容器，而是将pod移出endpoint，就绪探针确保了service中的pod都是可用的，确保客户端只与正常的pod交互并且客户端永远不会知道系统存在问题。

4、pod一致处于pending状态一般有哪些情况，怎么排查？（重点，持续更新）

答：一个pod一开始创建的时候，它本身就是会处于pending状态，这时可能是正在拉取镜像，正在创建容器的过程。
如果等了一会发现pod一直处于pending状态，那么我们可以使用kubectl describe命令查看一下pod的Events详细信息。一般可能会有这么几种情况导致pod一直处于pending状态：
1）调度器调度失败。Scheduer调度器无法为pod分配一个合适的node节点。而这又会有很多种情况，比如，node节点处在cpu、内存压力，导致无节点可调度；pod定义了资源请求，没有node节点满足资源请求；node节点上有污点而pod没有定义容忍；pod中定义了亲和性或反亲和性而没有节点满足这些亲和性或反亲和性；以上是调度器调度失败的几种情况。
2）pvc、pv无法动态创建。如果因为pvc或pv无法动态创建，那么pod也会一直处于pending状态，比如要使用StatefulSet 创建redis集群，因为粗心大意，定义的storageClassName名称写错了，那么会造成无法创建pvc，这种情况pod也会一直处于pending状态，或者，即使pvc是正常创建了，但是由于某些异常原因导致动态供应存储无法正常创建pv，那么这种情况pod也会一直处于pending状态。

5、简述Kubernetes ingress

- Kubernetes的Ingress资源对象，用于将不同URL的访问请求转发到后端不同的Service，以实现HTTP层的业务路由机制。
- Kubernetes使用了Ingress策略和Ingress Controller，两者结合并实现了一个完整的Ingress负载均衡器。使用Ingress进行负载分发时，Ingress Controller基于Ingress规则将客户端请求直接转发到Service对应的后端Endpoint（Pod）上，从而跳过kube-proxy的转发功能，kube-proxy不再起作用，全过程为：ingress controller + ingress 规则 ----> services。
- 同时当Ingress Controller提供的是对外服务，则实际上实现的是边缘路由器的功能。

## Reference

- [k8s面试题大全（持续更新中）](https://blog.csdn.net/MssGuo/article/details/125267817)
- [67道kubernetes面试题](https://github.com/kubernetes-cn/k8s-books/blob/main/k8s-book/67%E9%81%93kubernetes%E9%9D%A2%E8%AF%95%E9%A2%98.md)
