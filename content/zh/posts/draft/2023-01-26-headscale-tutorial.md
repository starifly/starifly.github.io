---
title: headscale使用教程
author: starifly
date: 2023-01-26T16:35:04+08:00
lastmod: 2023-01-26T16:35:04+08:00
categories: [headscale,网络]
tags: [headscale,tailscale,wireguard,网络]
draft: true
slug: headscale-tutorial
---

Headscale是一个开源的基于wireguard的控制服务，可以自己部署控制，功能相当于Tailscale的控制服务器，但是Tailscale的控制服务器是不开源的，而且对免费用户有诸多限制。

## Headscale 部署

Headscale 部署很简单，推荐直接在 Linux 主机上安装。

> 理论上来说只要你的 Headscale 服务可以暴露到公网出口就行，但最好不要有 NAT，所以推荐将 Headscale 部署在有公网 IP 的云主机上。

首先需要到其 GitHub 仓库的 Release 页面下载最新版的二进制文件。

```
$ wget --output-document=/usr/local/bin/headscale \
   https://github.com/juanfont/headscale/releases/download/v<HEADSCALE VERSION>/headscale_<HEADSCALE VERSION>_linux_<ARCH>

$ chmod +x /usr/local/bin/headscale
```

创建配置目录：

```
$ mkdir -p /etc/headscale
```

创建目录用来存储数据与证书：

```
$ mkdir -p /var/lib/headscale
```

创建空的 SQLite 数据库文件：

```
$ touch /var/lib/headscale/db.sqlite
```

创建 Headscale 配置文件：

```
wget https://github.com/juanfont/headscale/raw/main/config-example.yaml -O /etc/headscale/config.yaml
```

- 修改配置文件，将 server_url 改为公网 IP 或域名。如果是国内服务器，域名必须要备案。我的域名无法备案，所以我就直接用公网 IP 了。  
- 如果暂时用不到 DNS 功能，可以先将 magic_dns 设为 false。  
- 可自定义私有网段，也可同时开启 IPv4 和 IPv6：

```
ip_prefixes:
  # - fd7a:115c:a1e0::/48
  - 10.1.0.0/16
```

创建 SystemD service 配置文件：

```
# /etc/systemd/system/headscale.service
[Unit]
Description=headscale controller
After=syslog.target
After=network.target

[Service]
Type=simple
User=headscale
Group=headscale
ExecStart=/usr/local/bin/headscale serve
Restart=always
RestartSec=5

# Optional security enhancements
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/lib/headscale /var/run/headscale
AmbientCapabilities=CAP_NET_BIND_SERVICE
RuntimeDirectory=headscale

[Install]
WantedBy=multi-user.target
```

创建 headscale 用户：

```
$ useradd headscale -d /home/headscale -m
```

修改 /var/lib/headscale 目录的 owner：

```
$ chown -R headscale:headscale /var/lib/headscale
```

修改配置文件中的 unix_socket：

```
unix_socket: /var/run/headscale/headscale.sock
```

Reload SystemD 以加载新的配置文件：

```
$ systemctl daemon-reload
```

启动 Headscale 服务并设置开机自启：

```
$ systemctl enable --now headscale
```

查看运行状态：

```
$ systemctl status headscale
```

Tailscale 中有一个概念叫 tailnet，你可以理解成租户，租户与租户之间是相互隔离的。Headscale 也有类似的实现叫 namespace，即命名空间。我们需要先创建一个 namespace，以便后续客户端接入，例如：

```
$ headscale namespaces create default
```

查看命名空间：

```
$ headscale namespaces list

ID | Name | Created

1 | default | 2022-03-09 06:12:06
```

## Tailscale 客户端接入

Headscale 控制服务部署好后，需要用到Tailscale客户端去接入。

目前除了 iOS 客户端，其他平台的客户端都有办法自定义 Tailscale 的控制服务器。

|  OS   | 是否支持 Headscale  |
|  ----  | ----  |
| Linux  | Yes |
| OpenBSD  | Yes |
| macOS  | Yes |
| Windows  | Yes 参考[Windows 客户端文档](https://github.com/juanfont/headscale/blob/main/docs/windows-client.md) |
| Android  | 需要自己编译客户端 |
| ios  | 暂不支持 |

我们先来看下 Linux 平台的接入。

**Linux**

下载官方静态编译的二进制文件：

```
$ wget https://pkgs.tailscale.com/stable/tailscale_1.22.2_amd64.tgz
```

解压：

```
$ tar zxvf tailscale_1.22.2_amd64.tgz
x tailscale_1.22.2_amd64/
x tailscale_1.22.2_amd64/tailscale
x tailscale_1.22.2_amd64/tailscaled
x tailscale_1.22.2_amd64/systemd/
x tailscale_1.22.2_amd64/systemd/tailscaled.defaults
x tailscale_1.22.2_amd64/systemd/tailscaled.service
```

将二进制文件复制到官方软件包默认的路径下：

```
$ cp tailscale_1.22.2_amd64/tailscaled /usr/sbin/tailscaled
$ cp tailscale_1.22.2_amd64/tailscale /usr/bin/tailscale
```

将 systemD service 配置文件复制到系统路径下：

```
$ cp tailscale_1.22.2_amd64/systemd/tailscaled.service /lib/systemd/system/tailscaled.service
```

将环境变量配置文件复制到系统路径下：

```
$ cp tailscale_1.22.2_amd64/systemd/tailscaled.defaults /etc/default/tailscaled
```

启动 tailscaled.service 并设置开机自启：

```
$ systemctl enable --now tailscaled
```

Tailscale 接入 Headscale：

```
# 将 <HEADSCALE_PUB_IP> 换成你的 Headscale 公网 IP 或域名
$ tailscale up --login-server=http://<HEADSCALE_PUB_IP>:8080 --accept-routes=true --accept-dns=false
```

这里推荐将 DNS 功能关闭，因为它会覆盖系统的默认 DNS。如果你对 DNS 有需求，可自己研究官方文档，这里不再赘述。

执行完上面的命令后，会出现下面的信息：

```
To authenticate, visit:

http://xxxxxx:8080/register?key=905cf165204800247fbd33989dbc22be95c987286c45aac303393704

1150d846
```

在浏览器中打开该链接，将其中的命令复制粘贴到 headscale 所在机器的终端中，并将 NAMESPACE 替换为前面所创建的 namespace。

```
$ headscale -n default nodes register --key 905cf165204800247fbd33989dbc22be95c987286c45aac3033937041150d846
Machine register
```

注册成功，查看注册的节点：

```
$ headscale nodes list
```

回到 Tailscale 客户端所在的 Linux 主机，可以看到 Tailscale 会自动创建相关的路由表和 iptables 规则。路由表可通过以下命令查看：

```
$ ip route show table 52
```

查看 iptables 规则：

```
$ iptables -S
-P INPUT DROP
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-N ts-forward
-N ts-input
-A INPUT -j ts-input
-A FORWARD -j ts-forward
-A ts-forward -i tailscale0 -j MARK --set-xmark 0x40000/0xffffffff
-A ts-forward -m mark --mark 0x40000 -j ACCEPT
-A ts-forward -s 100.64.0.0/10 -o tailscale0 -j DROP
-A ts-forward -o tailscale0 -j ACCEPT
-A ts-input -s 10.1.0.5/32 -i lo -j ACCEPT
-A ts-input -s 100.115.92.0/23 ! -i tailscale0 -j RETURN
-A ts-input -s 100.64.0.0/10 ! -i tailscale0 -j DROP

$ iptables -S -t nat
-P PREROUTING ACCEPT
-P INPUT ACCEPT
-P OUTPUT ACCEPT
-P POSTROUTING ACCEPT
-A ts-postrouting -m mark --mark 0x40000 -j MASQUERADE
```

**macOS**

参考<https://fuckcloudnative.io/posts/how-to-set-up-or-migrate-headscale/#macos>

**Android**

参考<https://fuckcloudnative.io/posts/how-to-set-up-or-migrate-headscale/#android>

**Windows**

在浏览器中打开 URL：http://<HEADSCALE_PUB_IP>:8080/windows，然后按照提示就行操作，首先下载一个注册表，然后运行，再去[Official Windows Client](https://tailscale.com/download/windows)下载tailscale windows客户端并安装，之后在开始菜单打开并登陆（可以适用github账号单点登录）。

## 打通局域网

到目前为止我们只是打造了一个点对点的 Mesh 网络，各个节点之间都可以通过 WireGuard 的私有网络 IP 进行直连。但我们可以更大胆一点，还记得我在文章开头提到的访问家庭内网的资源吗？我们可以通过适当的配置让每个节点都能访问其他节点的局域网 IP。这个使用场景就比较多了，你可以直接访问家庭内网的 NAS，或者内网的任何一个服务。

假设你的家庭内网有一台 Linux 主机（比如 OpenWrt）安装了 Tailscale 客户端，我们希望其他 Tailscale 客户端可以直接通过家中的局域网 IP（例如 192.168.100.0/24） 访问家庭内网的任何一台设备。

配置方法很简单，首先需要设置 IPv4 与 IPv6 路由转发：

```
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding = 1' >> /etc/sysctl.conf
sysctl -p /etc/sysctl.conf
# IP伪装
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

客户端修改注册节点的命令，在原来命令的基础上加上参数 --advertise-routes=192.168.100.0/24。

```tailscale up --login-server=http://<HEADSCALE_PUB_IP>:8080 --accept-routes=true --accept-dns=false --advertise-routes=192.168.100.0/24
```

在 Headscale 端查看路由，可以看到相关路由是关闭的。

```
headscale nodes list|grep openwrt

6 | openwrt | [7LdVc] | default | 10.1.0.6 | false | 2022-03-20 15:50:46 | onlin

e | no

headscale routes list -i 6

Route | Enabled

192.168.100.0/24 | false
```

开启路由：

```
headscale routes enable -i 6 -r "192.168.100.0/24"

Route | Enabled

192.168.100.0/24 | true
```

其他节点查看路由结果：

```
ip route show table 52|grep "192.168.100.0/24"
192.168.100.0/24 dev tailscale0
```

现在你在任何一个 Tailscale 客户端所在的节点都可以 ping 通家庭内网的机器了，你在公司或者星巴克也可以像在家里一样用同样的 IP 随意访问家中的任何一个设备。

## 总结

目前从稳定性来看，Tailscale 比 Netmaker 略胜一筹，基本上不会像 Netmaker 一样时不时出现 ping 不通的情况，这取决于 Tailscale 在用户态对 NAT 穿透所做的种种优化，他们还专门写了一篇文章介绍[NAT 穿透的原理](https://tailscale.com/blog/how-nat-traversal-works/)，[中文版](https://arthurchiao.art/blog/how-nat-traversal-works-zh/)翻译自国内的 eBPF 大佬赵亚楠。

## Reference

- [Tailscale 基础教程：Headscale 的部署方法和使用教程](https://fuckcloudnative.io/posts/how-to-set-up-or-migrate-headscale/)
- [TailScale 实现「出口节点」Exit Node（导向所有流量经这出口节点）](https://blog.csdn.net/sillydanny/article/details/120653504)
