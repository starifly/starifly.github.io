---
title: kubeadm部署高可用k8s集群
author: starifly
date: 2020-12-04T23:09:41+08:00
lastmod: 2020-12-04T23:09:41+08:00
categories: [k8s]
tags: [k8s,docker,kubeadm,keepalived,haproxy]
draft: true
slug: kubeadm-deploying-ha-k8s-cluster
---

## 部署架构

![](/images/k8s-ha-01.png)

三个 master 节点 和 三个 node 节点，haproxy 用于203、204、205 三个 master 的 apiserver 的负载均衡，外部 kubectl 以及 nodes 访问 apiserver 的时候就可以用过keepalived 的虚拟ip(192.168.5.123)以及 haproxy 端口(6444)访问 master 集群的 apiserver。

## 环境说明

- Linux版本：CentOS 7.8.2003

- docker版本：1.13.1

- k8s版本：v1.19.4

## 安装前准备

**配置所有节点hosts**

```
# 需要先修改每台节点的 hostname
192.168.5.203 203
192.168.5.204 204
192.168.5.205 205
192.168.4.203 n203
192.168.4.204 n204
192.168.4.205 n205
```

**关闭各个节点防火墙和SElinux**

**关闭每台服务器swap分区**

**同步每台节点的时间**

**内核调整**

```
# 每台服务器内核调整,将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1
# 生效
sysctl -p
```

做完所有变更后重启各个主机。

> 安装或者初始化过程中，要及时使用tailf /var/log/messages或者journalctl -xeu kubelet命令查看日志是否有异常

## docker 安装

所有节点安装docker-ce，安装前先去GitHub上的k8s仓库，通过changlog确认各k8s所支持的docker版本

```
yum install docker
systemctl start docker && systemctl enable docker

# 确认docker运行状态
docker info
```

## kubernetes 安装

所有节点安装kubeadm，yum install kubeadm，kubelet和kubectl会作为依赖安装

```
yum install kubeadm
# 设置cgroup驱动和镜像仓库
vim /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=cgroupfs --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"

# 启动kubelet并设置开机启动
systemctl daemon-reload
systemctl start kubelet && systemctl enable --now kubelet
```

## haproxy、keepalived安装

haproxy、keepalived只需要在三个master节点上安装

```
yum install haproxy keepalived -y
```

**haproxy配置**

```
cat /etc/haproxy/haproxy.cfg

global
log 127.0.0.1 local0
log 127.0.0.1 local1 notice
maxconn 4096
#chroot /usr/share/haproxy
#user haproxy
#group haproxy
daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    retries 3
    option redispatch
    timeout connect  5000
    timeout client  50000
    timeout server  50000

frontend stats-front
  bind *:8081
  mode http
  default_backend stats-back

frontend fe_k8s_6444
  bind *:6444
  mode tcp
  timeout client 1h
  log global
  option tcplog
  default_backend be_k8s_6443
  acl is_websocket hdr(Upgrade) -i WebSocket
  acl is_websocket hdr_beg(Host) -i ws

backend stats-back
  mode http
  balance roundrobin
  stats uri /haproxy/stats
  stats auth pxcstats:secret

backend be_k8s_6443
  mode tcp
  timeout queue 1h
  timeout server 1h
  timeout connect 1h
  log global
  balance roundrobin
  server 203 192.168.5.203:6443
  server 204 192.168.5.204:6443
  server 205 192.168.5.205:6443

# 启动haproxy并设置开机启动
systemctl daemon-reload
systemctl start haproxy && systemctl enable --now haproxy
```

**keepalived配置**

```
cat /etc/keepalived/keepalived.conf

! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
   script_user root
   enable_script_security
}

vrrp_script check_haproxy {
    script "/bin/bash -c 'if [[ $(netstat -nlp | grep 6444) ]]; then exit 0; else exit 1; fi'"
    interval 3
    weight -51
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state MASTER
    interface ens192
    virtual_router_id 51
    priority 250
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 35f18af7190d51c9f7f78f37300a0cbd
    }
    virtual_ipaddress {
        192.168.5.123
    }
    track_script {
        check_haproxy
    }
}

# 启动keepalived并设置开机启动
systemctl daemon-reload
systemctl start keepalived && systemctl enable --now keepalived
```

说明：

- master-0节点为***MASTER***，其余节点为***BACKUP***
- priority各个几点到优先级相差50，范围：0～250（非强制要求）

## 第一个master节点初始化

通过配置文件初始化，也可以通过命令行的方式，但是推荐使用配置文件，便于后期维护管理。

``` yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.19.4
imageRepository: daocloud.io/daocloud
controlPlaneEndpoint: 192.168.5.123:6444
apiServer:
  timeoutForControlPlane: 4m0s
  certSANs:
  - 192.168.5.203
  - 192.168.5.204
  - 192.168.5.205
networking:
  serviceSubnet: 192.168.0.0/16
  podSubnet: "10.244.0.0/16"
scheduler: {}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs
```

```
kubeadm init --config=kubeadm-config.yaml --upload-certs

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**安装calico网络**

```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
```

修改pod网段为之前kubeadm init初始化的pod网段

```
- name: CALICO_IPV4POOL_CIDR
  value: "10.244.0.0/16"
```

```
kubectl apply -f calico.yaml
```

查询状态，等待所有pod都就位

```
kubectl get nodes
kubectl get po -n kube-system
```

## 初始化第二、三个master节点

把主节点的证书复制到其它master节点，先在第二、三个master创建相应目录

```
mkdir -p /etc/kubernetes/pki/etcd/
```

拷贝第一个master节点至第二、三个master节点

```
scp /etc/kubernetes/pki/ca.* 192.168.5.204:/etc/kubernetes/pki/
scp /etc/kubernetes/pki/sa.* 192.168.5.204:/etc/kubernetes/pki/
scp /etc/kubernetes/pki/front-proxy-ca.* 192.168.5.204:/etc/kubernetes/pki/
scp /etc/kubernetes/pki/etcd/ca.* 192.168.5.204:/etc/kubernetes/pki/etcd/
scp /etc/kubernetes/admin.conf 192.168.5.204:/etc/kubernetes/
```

第二、三master节点加入集群

```
kubeadm join 192.168.5.123:6444 --token e2qedt.44qs629b3o76jttw     --discovery-token-ca-cert-hash sha256:4c03e1cf94b2c84b3e19dd61d17c6051d2555733a126dd26bdc4d65a6d7a9d19     --control-plane

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 查看节点状态
kubectl get nodes -o wide
```

## 初始化node节点

把主节点admin.conf证书复制到其他三个node节点

```
scp /etc/kubernetes/admin.conf 192.168.4.203:/etc/kubernetes/
scp /etc/kubernetes/admin.conf 192.168.4.204:/etc/kubernetes/
scp /etc/kubernetes/admin.conf 192.168.4.205:/etc/kubernetes/
```

执行初始化

```
kubeadm join 192.168.5.123:6444 --token e2qedt.44qs629b3o76jttw     --discovery-token-ca-cert-hash sha256:4c03e1cf94b2c84b3e19dd61d17c6051d2555733a126dd26bdc4d65a6d7a9d19
```

## 其它

- 由于master和etcd是部署在一起，而etcd选举要过半数，所以不能超过半数的master节点宕机
- 初始化失败，要使用kubeadm reset重置后，再重新初始化

## Reference
- [kubeadm部署k8s高可用集群](https://www.cnblogs.com/lfl17718347843/p/13417304.html)
- [基于kubeadm的kubernetes高可用集群部署](https://my.oschina.net/u/3133713/blog/1068894)
- [k8s master节点的高可用 (haproxy + keepalived)](https://www.cnblogs.com/cjwnb/p/12532402.html)
