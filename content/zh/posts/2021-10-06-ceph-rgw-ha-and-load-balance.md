---
title: ceph rgw高可用和负载均衡
author: starifly
date: 2021-10-06T19:46:05+08:00
lastmod: 2021-10-06T19:46:05+08:00
categories: [ceph]
tags: [ceph]
draft: false
slug: ceph-rgw-ha-and-load-balance
---

## 环境配置

安装依赖

`yum install pcre-devel.x86_64 openssl-devel   zlib-devel  systemd-devel gcc gcc-c++`

内核调优

```bash
#vim /etc/sysctl.conf

vm.swappiness = 0
net.ipv4.ip_forward = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 1025 65023
net.ipv4.tcp_max_syn_backlog = 10240
```

检查

```bash
/usr/sbin/sysctl net.ipv4.ip_nonlocal_bind
/usr/sbin/sysctl net.ipv4.ip_forward
cat /proc/sys/net/ipv4/ip_forward
```

> 查看是否开启了ip转发功能
> 如果上述文件中的值为0,说明禁止进行IP转发；如果是1,则说明IP转发功能已经打开。

## haproxy

下载安装包

`wget https://src.fedoraproject.org/repo/pkgs/haproxy/haproxy-2.0.1.tar.gz/sha512/bcc2c6fa1fe5699f110a2b2ce5abcec7f4ebff94a2850d731f6d86aadeb7f4048b6f372db6724a91191c2ecc2853f5ac576233e0ff84ffef3de6c80d1250f1b6/haproxy-2.0.1.tar.gz`

解压

`tar -zxvf haproxy-2.0.1.tar.gz`

开始编译安装

```bash
cd haproxy-2.0.1/
# 可指定安装目录PREFIX=/usr/local/haproxy，默认为/usr/local/sbin/
make TARGET=linux-glibc   USE_OPENSSL=1 USE_SYSTEMD=1 USE_PCRE=1  USE_ZLIB=1  && make install
```

注册到系统服务

`vim /usr/lib/systemd/system/haproxy.service`

```cfg
[Unit]
Description=HAProxy Load Balancer
After=syslog.target network.target

[Service]
ExecStartPre=/usr/local/sbin/haproxy -f /etc/haproxy/haproxy.cfg   -c -q
ExecStart=/usr/local/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg  -p /run/haproxy.pid
ExecReload=/bin/kill -USR2 $MAINPID

[Install]
WantedBy=multi-user.target
```

修改配置

`vim /etc/haproxy/haproxy.cfg`

```cfg
global
log 127.0.0.1 local0
log 127.0.0.1 local1 notice
maxconn 4096
# 注 下面完成了工作进程数及亲缘性绑定，若想做的极致一点还可以通过在内核启动文件即 grub 文件启动参数来实现 内核进程绑定以及进程隔离
nbproc 3 # 配置进程数，不包含 status
cpu-map 1 1 # 亲缘性绑定。前一个数字是进程号，后一个数字是CPU内核的号，注意将0号cpu留给内核
cpu-map	2 2
cpu-map	3 3
stats bind-process 3
#chroot /usr/share/haproxy
#user haproxy
#group haproxy
daemon

defaults
  mode                    http
  log                     global
  option                  httplog
  option                  dontlognull
  option http-server-close
  option forwardfor       except 127.0.0.0/8
  option                  redispatch
  retries                 3
  timeout http-request    10s
  timeout queue           1m
  timeout connect         10s
  timeout client          1m
  timeout server          1m
  timeout http-keep-alive 10s
  timeout check           10s

frontend http_web
  bind *:2003
  mode tcp
  #timeout client 1h
  #log global
  #option tcplog
  default_backend rgw
  #acl is_websocket hdr(Upgrade) -i WebSocket
  #acl is_websocket hdr_beg(Host) -i ws

backend rgw
  mode tcp
  #timeout queue 1h
  #timeout server 1h
  #timeout connect 1h
  #log global
  balance roundrobin
  #hash-type consistent
  server ceph001 192.168.5.203:20003 check inter 3s fall 2 rise 2
  server ceph002 192.168.5.204:20003 check inter 3s fall 2 rise 2
  server ceph003 192.168.5.205:20003 check inter 3s fall 2 rise 2
```

启动

`systemctl start haproxy && systemctl enable haproxy`

## keepalived

下载安装包

`wget --no-check-certificate http://www.keepalived.org/software/keepalived-1.2.22.tar.gz`

解压

`tar -zxvf keepalived-1.2.22.tar.gz`

编译安装

```bash
cd keepalived-1.2.22
./configure --prefix=/opt/keepalived
make && make install
cp /opt/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/
cp /opt/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
mkdir /etc/keepalived
cp /opt/keepalived/etc/keepalived/keepalived.conf /etc/keepalived
cp /opt/keepalived/sbin/keepalived /usr/sbin/
```

修改配置

> 全部节点采用backup且priority值一样，为了减少VIP来回切换。

`vim /etc/keepalived/keepalived.conf`

```cfg
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
}

vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 2
    weight -2
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens192
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 35f18af7190d51c9f7f78f37300a0cbd
    }
    virtual_ipaddress {
        192.168.5.202
    }
    track_script {
        check_haproxy
    }
}
```

启动

```bash
/etc/init.d/keepalived start
systemctl enable keepalived
```

## Reference

- [配置Ceph RGW (对象存储网关) 的高可用和负责均衡](https://mp.weixin.qq.com/s/YSxzr4jNeyfQUWebuY38TA)
- [HAproxy总结](https://www.jianshu.com/p/a99d3a124883)
- [haproxy2.0-编译安装-centos7.6](https://www.cnblogs.com/66li/p/12058774.html)
- [l4 l7 代理_Haproxy-4层和7层代理负载实战](https://blog.csdn.net/weixin_33417703/article/details/112949909)
- [调优 Haproxy 代理 Ceph](https://www.bianchengquan.com/article/431961.html)
