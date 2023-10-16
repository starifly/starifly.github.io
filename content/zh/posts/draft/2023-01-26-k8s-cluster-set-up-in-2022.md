---
title: 2022版使用云服务器搭建公网k8s容器集群
author: starifly
date: 2023-01-26T14:47:14+08:00
lastmod: 2023-01-29
categories: [k8s]
tags: [k8s]
draft: true
slug: k8s-cluster-set-up-in-2022
---

## 总体流程一览

主要流程如下：  
1.准备云主机，升级CentOS系统到7.9  
2.所有节点上安装Docker和Kubeadm，拉取相关镜像  
3.在Master节点初始化集群，包括kubectl和部署CN容器网络插件  
4.把Node节点加入k8s集群

可视化界面和私有镜像仓库请参考其他文章：  
1.部署Dashboard Web 页面，可视化查看Kubernetes资源，看我下一篇文章：k8s dashboard安装  
2.部署Harbor私有仓库，存放镜像资源（非必要，省略介绍）  

下面，开始介绍各流程详细配置步骤。

## 环境准备

三台腾讯云主机，node1作为master节点，另外两台作为work节点，centos为CentOS Stream release 8（如果是7版本，必须高于7.9），三台节点均安装docker、kubeadm、kubelet、kubectl、flannel。

## 所有节点CentOS设置

### 基础设置

可以创建 k8s-pre-install-centos.sh 脚本，一键设置：

```
$ vim k8s-pre-install-centos.sh

#!/bin/sh

function set_base(){
  # 关闭防火墙，PS：如果使用云服务器，还需要在云服务器的控制台中把防火墙关闭了或者允许所有端口。
  systemctl stop firewalld
  systemctl disable firewalld

  # 关闭SELinux，这样做的目的是：为了让容器能读取主机文件系统。
  setenforce 0

  # 永久关闭swap分区交换，kubeadm规定，一定要关闭
  swapoff -a
  sed -ri 's/.*swap.*/#&/' /etc/fstab

  # iptables配置
  cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

  # iptables生效参数
  sysctl --system
}

set_base
```

执行上述脚本：

```
chmod 777 k8s-pre-install-centos.sh && ./k8s-pre-install-centos.sh
```

### 主机名设置

三台节点修改主机名并配置hosts文件

## 所有节点安装Docker

k8s支持 3种容器运行时，这里我们优先使用熟悉的 Docker 作为容器运行时。

### 配置Docker守护程序

```
$ sudo mkdir /etc/docker
$ cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "registry-mirrors": ["https://6ijb8ubo.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

- registry-mirrors：镜像加速。
- cgroupdriver：使用systemd。
- log-driver：使用json日志，大小为100m。

启动docker并确认Cgroup Driver为systemd

```
$ docker info | grep "Cgroup Driver"
 Cgroup Driver: systemd
```

> PS：因为k8s是默认systemd作为cgroup driver，如果Docker使用另外的驱动，则可能出现不稳定的情况。

## 所有节点安装kubeadm

为保证不过期，请最终以 官方文档 为主：

### 配置yum源（使用aliyun，google你知道的）

```
$ cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

$ yum makecache # 更新yum
```

### 安装kubeadm, kubelet, kubectl

$ sudo yum install -y kubelet-1.23.17 kubeadm-1.23.17 kubectl-1.23.17 --disableexcludes=kubernetes

启动klubelet，并设置为开机启动

```
$ sudo systemctl start kubelet
$ sudo systemctl enable kubelet
```

## 所有节点拉取Docker镜像

> ps：该1.23.17版本，需要docker为20，超过的不保证成功
拉取Docker镜像

### 查看初始化需要的镜像

```
$ kubeadm config images list

k8s.gcr.io/kube-apiserver:v1.23.17
k8s.gcr.io/kube-controller-manager:v1.23.17
k8s.gcr.io/kube-scheduler:v1.23.17
k8s.gcr.io/kube-proxy:v1.23.17
k8s.gcr.io/pause:3.6
k8s.gcr.io/etcd:3.5.1-0
k8s.gcr.io/coredns/coredns:v1.8.6
```

### 替换k8s镜像源

k8s模式镜像仓库是 k8s.gcr.io，由于众所周知的原因是无法访问的。

故这里需要创建配置 kubeadm-config-image.yaml 替换成阿里云的源：

```
$ vim kubeadm-config-image.yaml

apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
# 默认为k8s.gcr.io，但是网络不通，所以要替换为阿里云镜像
imageRepository: registry.aliyuncs.com/google_containers 
```

### 确认镜像仓库改变

```
$ kubeadm config images list --config kubeadm-config-image.yaml

registry.aliyuncs.com/google_containers/kube-apiserver:v1.23.17
registry.aliyuncs.com/google_containers/kube-controller-manager:v1.23.17
registry.aliyuncs.com/google_containers/kube-scheduler:v1.23.17
registry.aliyuncs.com/google_containers/kube-proxy:v1.23.17
registry.aliyuncs.com/google_containers/pause:3.6
registry.aliyuncs.com/google_containers/etcd:3.5.1-0
registry.aliyuncs.com/google_containers/coredns:v1.8.6
```

### 拉取镜像

在 所有机器 上执行，把这些镜像提前拉好。

```
$ kubeadm config images pull --config kubeadm-config-image.yaml
```

## Master节点初始化集群

### 生成默认配置 kubeadm-config.yaml

```
kubeadm config print init-defaults > kubeadm-config.yaml
```

```
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.0.4.7 # 指定master节点的IP地址（这里还是要用内网地址）
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  imagePullPolicy: IfNotPresent
  name: master01  # 改成master的主机名
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers  # 默认为k8s.gcr.io，但是网络不通，所以要替换为阿里云镜像
kind: ClusterConfiguration
kubernetesVersion: 1.23.17  # 指定kubernetes版本号，使用kubeadm config print init-defaults生成的即可
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16  # 指定pod网段，10.244.0.0/16用于匹配flannel默认网段
scheduler: {}
```

### 检查环境

```
$ kubeadm init phase preflight --config=kubeadm-config.yaml
```

这个命令会检查配置文件是否正确，以及系统环境是否支持kubeadm的安装。

### 初始化kubeadm集群

只需要在master上执行如下命令：

```
$ kubeadm init --config=kubeadm-config.yaml
```

到这里，会有2种结果：

- 如果是内网，上面的docker版本，kubeadm版本没错的话，会成功，直接跳到4步骤。
- 如果在云服务器（腾讯云，阿里云）上，一定会失败（原因和办法在这里）：

```
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests”
// ...
[kubelet-check] Initial timeout of 40s passed.
```

### 云服务器初始化失败解决版本

1）编辑etcd配置文件

配置文件位置：/etc/kubernetes/manifests/etcd.yaml，改成：

```
    - --listen-client-urls=https://127.0.0.1:2379
    - --listen-peer-urls=https://127.0.0.1:2380
```

> 引用 在腾讯云安装K8S集群 ：
此处"118.195.137.68"为腾讯云公网ip，要关注的是"–listen-client-urls"和"–listen-peer-urls"。需要把–listen-client-urls后面的公网IP删除，把–listen-peer-urls改成127.0.0.1：2380
原因是因为腾讯云只要选择VPC网络均是采用NAT方式将公网IP映射到私人网卡的，有兴趣的同学可以了解下NAT。这也就是为什么很多同事无法在腾讯云或阿里云上安装k8s集群的原因

2）重新初始化，但是跳过etcd文件已经存在的检查：

```
# 重新初始化，跳过配置文件生成环节，不要etcd的修改要被覆盖掉
$ kubeadm init --config=kubeadm-config.yaml --skip-phases=preflight,certs,kubeconfig,kubelet-start,control-plane,etcd
```

成功初始化后，按照输出信息操作接下来的步骤即可。

### 配置kubectl（master）

拷贝配置文件到kubectl默认加载路径：

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

使用kubectl查看集群信息，在Master节点上执行，输出集群信息：

```
$ kubectl get nodes
NAME       STATUS     ROLES                  AGE   VERSION
master01   NotReady   control-plane,master   15m   v1.23.17

$ kubectl get cs
etcd-0               Healthy   {"health":"true","reason":""}
controller-manager   Healthy   ok
scheduler            Healthy   ok
```

### 安装CN网络（master）

```
$ curl https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml>>kube-flannel.yml
$ chmod 777 kube-flannel.yml 
$ kubectl apply -f kube-flannel.yml
```

等待几分钟，再查看Master节点状态，由NotRead变成了Ready状态

### 允许master节点部署pod

去除污点即可

## 把Node节点加入集群

在node上执行上面kubeadm输出的命令

## 清理环境

如果你的集群安装过程中遇到了其他问题，我们可以使用下面的命令来进行重置：

```
# 所有节点
$ kubeadm reset
$ systemctl stop kubelet
$ yum remove kubeadm kubectl kubelet kubernetes-cni kube*
$ yum autoremove
$ ifconfig cni0 down && ip link delete cni0
$ ifconfig flannel.1 down && ip link delete flannel.1
$ rm -rf /var/lib/cni/
$ rm -rf ~/.kube
$ rm -rf /var/lib/kubelet/*
$ rm -rf /etc/kubernets/
$ rm -f /run/flannel/subnet.env
```

## Reference

- [2022版使用云服务器搭建公网k8s容器集群](http://www.yaotu.net/biancheng/14301.html)
- [在腾讯云安装K8S集群](https://blog.51cto.com/u_15152259/2690063)
