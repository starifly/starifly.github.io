---
title: cc系统的keepalived配置
author: starifly
date: 2020-12-04T23:24:53+08:00
lastmod: 2020-12-04T23:24:53+08:00
categories: [keepalived]
tags: [keepalived]
draft: true
slug: cc-linkserver-keepalived
---

keepalived配置

```
! Configuration File for keepalived
global_defs {
    router_id node1
}

vrrp_script check_ng {
	script "/etc/keepalived/check_nginx.sh"
	interval 5
	weight -20
	#fall 3
	rise 1
}
vrrp_script check_lk {
	script "/etc/keepalived/check_link.sh"
	interval 10
	weight -20
	#fall 3
	rise 1
}

vrrp_instance VI_1 {
    state MASTER      # 初始化设置为MASTER节点
    interface eno16780032
    virtual_router_id 50   # 同一个VRRP实例中每个节点的虚拟路由ID必须相同
    priority 100             # MASTER节点的优先级必须高于BACKUP节点
    advert_int 1
    authentication {
        auth_type PASS    # 验证类型为密码认证
        auth_pass 1111 # 验证密码，需要注意密码最大长度为8位
    }
    virtual_ipaddress {
        192.168.5.226 dev eno16780032  # 主备节点的VIP一定要相同
    }
    track_script {
        check_ng
	check_lk
    }
}
```

检测nginx的脚本

```
#!/bin/sh
if [ $(ps -C nginx --no-header | wc -l) -eq 0 ]; then
    systemctl start nginx
fi
sleep 1

if [ $(ps -C nginx --no-header | wc -l) -eq 0 ]; then
    exit 1
else
    exit 0
fi
```

检测LinkServer脚本

```
#!/bin/sh
if [ $(ps -C LinkServer --no-header | wc -l) -eq 0 ]; then
    systemctl start link
    sleep 8
fi

if [ $(ps -C LinkServer --no-header | wc -l) -eq 0 ]; then
    exit 1
else
    exit 0
fi
```
