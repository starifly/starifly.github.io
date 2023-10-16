---
title: freeswitch被攻击的限制措施
author: starifly
date: 2019-12-28T20:40:11+08:00
categories: [freeswitch,网络]
tags: [freeswitch,网络,防火墙,ipset,Fail2Ban]
draft: true
slug: ipset-to-limit-freeswitch-attacks
---

## ipset

**安装ipset和ipset服务**

`yum install ipset ipset-service`

**建表**

`ipset create china hash:net hashsize 10000 maxelem 1000000`

**批量增加中国IP到ipset的china表**

```bash
#!/bin/bash
rm -f cn.zone
wget http://www.ipdeny.com/ipblocks/data/countries/cn.zone
for i in `cat cn.zone`
do
    ipset add china $i 
done
```

以上内容保存成脚本执行即可

查看一下：`ipset list china`

**持久化 ipset**

`service ipset save`

然后还得修改配置文件 `/etc/sysconfig/ipset-config` ，把其中的“no”改为“yes”

除了用服务持久化，也可以手动保存： `ipset save > /etc/sysconfig/ipset`

**增加防火墙规则**

```conf
-A INPUT -s 192.168.5.0/24 -p udp -m state --state NEW -m udp --dport 5090 -j ACCEPT # 内网段允许访问内线端口
-A INPUT -m set --match-set china src -p udp -m udp --dport 5078 -j ACCEPT # 外线端口只允许中国 IP 访问
```

## 修改外联模式相关配置

修改文件 `conf/autoload_configs/event_socket.conf.xml` 中的端口和密码等，如果此处修改了密码 cti 配置文件中的密码也应同步修改。

## SIP scanner iptables block

```shell
iptables -I INPUT -j DROP -p udp --dport 5060 -m string --string "VaxSIPUserAgent" --algo bm
iptables -I INPUT -j DROP -p udp --dport 5060 -m string --string "friendly-scanner" --algo bm
iptables -I INPUT -j DROP -p udp --dport 5060 -m string --string "sipcli" --algo bm
iptables -I INPUT -j DROP -p udp --dport 5080 -m string --string "VaxSIPUserAgent" --algo bm
iptables -I INPUT -j DROP -p udp --dport 5080 -m string --string "friendly-scanner" --algo bm
iptables -I INPUT -j DROP -p udp --dport 5080 -m string --string "sipcli" --algo bm
```

以下这六个规则将阻止随机扫描Internet的所有sip扫描程序流量的绝大多数。 将这些规则与Fail2Ban结合使用，您将处于良好状态以避免恶意攻击者。

## Reference

- [freeswitch被外国IP攻击盗打的防护措施](https://blog.csdn.net/HanJiezZ/article/details/88570425)
- [freeswitch安装后马上修改默认密码限攻击](https://blog.csdn.net/dws123dws/article/details/90340175)
- [Linux iptables 扩展 ipset 使用教程](https://www.zhoutao.org/blog/2017/11/757.html)
- [Firewall - FreeSWITCH - Confluence](https://www.freeswitch.org/confluence/display/FREESWITCH/Firewall)
- [CentOS7下安装和使用Fail2ban](https://www.jianshu.com/p/4fdec5794d08)
- [freeswitch Can't find user](http://www.pianshen.com/article/3765360400/)
- [FreeSWITCH配置fail2ban拦载一般的恶意骚扰](http://freeswitch.net.cn/23.html)
