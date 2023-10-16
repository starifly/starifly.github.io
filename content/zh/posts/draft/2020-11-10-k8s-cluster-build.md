---
title: k8s集群搭建
author: starifly
date: 2020-11-10T21:24:34+08:00
lastmod: 2020-11-10T21:24:34+08:00
categories: [k8s]
tags: [k8s,docker]
draft: true
slug: k8s-cluster-build
---

## 环境介绍

使用两台虚拟机，一台master节点，一台业务节点，如果可以，多创建几台业务节点也可以，安装部署方法等同。

系统要求 :

1、Ubuntu16.04+/CentOS7

2、内存要大于等于 2 G

3、CPU 核数大于等于 2 核

4、集群中的所有机器的网络彼此均能相互连接(公网和内网都可以)

5、节点之中不可以有重复的主机名、MAC 地址或 product_uuid

6、开启机器上的某些端口

7、禁用交换分区，为了保证 kubelet 正常工作

软件安装：

链接：https://pan.baidu.com/s/1BvrQpFGWPKOJB7Z82nHalA， 密码：4m9z，个人百度网盘也保存了。

1、Master节点：

主机名：Master

两块网卡：

地址：10.0.3.15（公网）192.168.56.104（私网）

2、Minion-1节点：

主机名：Minion1

两块网卡：

地址：10.0.3.16（公网）192.168.56.105（私网）

 

软件版本：

kubernetes：v1.9.0

docker：17.03

etcd:3.1.10

pause :3.0

flannel:v0.9.1

kubernetes-dashboard:v1.8.1

 

kubeadm默认要从google的镜像仓库下载镜像，我们将附件中镜像文件导入到master节点和minion节点上。

文件名：k8s_images.tar.bz2

MD5： b60ad6a638eda472b8ddcfa9006315ee

## 准备工作

1、配置vm1和vm2节点ssh互信。（master和业务节点同步执行）

```
# ssh-keygen

# ssh-copy-id -i /root/.ssh/id_rsa.pub  root@vm2

 

# ssh-keygen

# ssh-copy-id -i /root/.ssh/id_rsa.pub root@vm1

2、关闭防火墙和selinux

# systemctl stop firewalld && systemctl disable firewalld

# vi /etc/selinux/config

SELINUX=disabled //改完使用getenforce，如果显示未生效，需重启。

# echo "

net.bridge.bridge-nf-call-ip6tables = 1

net.bridge.bridge-nf-call-iptables = 1

" >> /etc/sysctl.conf

# sysctl -p
```

禁用selinux，主要为了允许容器可以访问主机文件系统和pod networks的需要。

设置内核参数主要是为了避免 RHEL/CentOS 7系统下出现路由异常。

3、设置各节点的主机名

[root@Master ~]# hostname

Master

[root@Minion1 ~]# hostname

Minion1

## 安装docker

安装 17.03.2-ce版本的docker，并导入image文件。（master和业务节点上同步执行）

```
# yum install bzip2

# tar -xjvf k8s_images.tar.bz2

# cd k8s_images

# yum -y localinstall docker-ce-*

# systemctl start docker && systemctl enable docker

# docker version

 

# cd k8s_images/docker_images/

# for i in $(ls *.tar);do docker load < $i;done

# cd ..

# docker load < kubernetes-dashboard_v1.8.1.tar

# docker images | grep google
```

## 安装Kubernetes

1、安装k8s 1.9.0版本软件包（master和业务节点上同步执行）

```
# cd /root/k8s_images/

# rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm

# rpm -ivh kube*.rpm

# rpm -qa |grep kube & rpm -qa |grep socat
```

保证以上包能正确安装。截图如下：

 

启动kubelet服务

systemctl enable kubelet && systemctl start kubelet

2、初始化master节点。（master节点上执行）

2.1 改驱动

kubelet默认的cgroup的driver和docker的不一样，docker默认的cgroupfs，kubelet默认为systemd，因此我们要修改成一致。在虚拟机上部署k8s 1.9版本需要关闭操作系统交换分区。

```
# swapoff -a

# grep -i 'cgroupfs' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"

# systemctl daemon-reload
```

2.2 初始化节点

初始化命令：

```
#kubeadm init --apiserver-advertise-address=192.168.56.104  --pod-network-cidr=10.244.0.0/16 --kubernetes-version=v1.9.0

//此处IP为master上私网IP地址。10.244.0.0/16地址可改可以不改，该地址为节点上pod见通信所用网段地址，如果改，需要将所改网段与kube-flannel.yml中地址保持一致，实验阶段可以先不改。

当看到如下提示即可：

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube

  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:

  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each nodeas root:

  kubeadm join --token 20049e.19abe8bacc412b0a 192.168.56.104:6443 --discovery-token-ca-cert-hash sha256:b44f687a629fe0d56a6700f8e6bbee1837190a64baad0ea057070e30c6a28142

# echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile

# source  ~/.bash_profile

//添加环境变量。

# kubectl version

如果初始化失败需要重新进行初始化，需要先进行reset一下

# kubeadm reset
```

2.3部署网络插件flannel

```
# wget https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml

# kubectl create -f kube-flannel.yml

clusterrole "flannel" created

clusterrolebinding "flannel" created

serviceaccount "flannel" created

configmap "kube-flannel-cfg" created

daemonset "kube-flannel-ds" created

 

如果报错的话：[root@k8s-master k8s_images]# kubectl create -f kube-flannel.yml

The connection to the server localhost:8080 was refused - did you specify the right host or port?

# 重新进行初始化，并且source bash_file
```
 

3、初始化业务节点。（业务节点上执行）

3.1 改驱动

```
# swapoff  -a

# grep -i 'cgroupfs' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"
```

3.2 初始化节点

```
# kubeadm join --token 20049e.19abe8bacc412b0a 192.168.56.104:6443 --discovery-token-ca-cert-hash sha256:b44f687a629fe0d56a6700f8e6bbee1837190a64baad0ea057070e30c6a28142
```

出现以下即可：

[discovery] Successfully established connection with API Server "192.168.56.104:6443"

 

This node has joined the cluster:

* Certificate signing request was sent to master and a response

  was received.

* The Kubelet was informed of the new secure connection details.

 

Run 'kubectl get nodes' on the master to see this node join the cluster.

4、查看初始化是否完成。（master节点上执行）

```
# systemctl daemon-reload

# systemctl restart kubelet

# kubectl get node

# kubectl get pod --all-namespaces
```

## 常见问题

1、如果是用虚拟机做实验，重启虚拟机后，master节点上出现：

解决方法：

```
# swapoff -a //关闭操作系统交换分区即可。具体原因尚不得知，还在研究中。
```

2、业务节点notready。

现象：

 

解决方法：

方法一：在master和业务节点上重启kubelet。

```
# swapoff -a

# systemctl restart kubelet
```
 

方法二：如果方法一无法达到效果，可以快速初始化节点：

步骤一：在master上：

```
# kubadm token list
```

获取token。

 

步骤二：在业务节点上：

```
# swapoff -a

# kubeadm reset

# kubeadm join --token 259ae3.7b3c1269c8dfb568 192.168.56.104:6443 --discovery-token-unsafe-skip-ca-verification
```

看到如下即可：

 

步骤三：在master上：

```
# systemctl daemon-reload

# systemctl restart kubelet
```

## Reference

- [2018-4-18 k8s安装部署过程](https://www.cnblogs.com/yue-hong/p/8894033.html)
