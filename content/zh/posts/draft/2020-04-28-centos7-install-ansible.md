---
title: centos7安装ansible
author: starifly
date: 2020-04-28T20:58:05+08:00
lastmod: 2020-04-28T20:58:05+08:00
categories: [ansible]
tags: [ansible,centos,linux]
draft: true
slug: centos7-install-ansible
---

1.确保机器上安装的是 Python 2.6 或者 Python 2.7 版本

2.查看yum仓库中是否存在ansible的rpm包，`yum list|grep ansible`

3.安装ansible服务： `yum install ansible -y`

4.修改ansible配置和主机列表hosts

1)关闭第一次使用ansible连接客户端时输入命令提示：

`sed -i "s@\#host_key_checking = False@host_key_checking = False@g" /etc/ansible/ansible.cfg`

指定日志路径：

`sed -i "s@\#log_path = \/var\/log\/ansible.log@log_path = \/var\/log\/ansible.log@g" /etc/ansible/ansible.cfg`

2)将所有主机ip加入到/etc/ansible/hosts文件中：

```
[app]
192.168.5.131
```

默认ssh的端口为22端口，如果为其他端口号，可在主机名后面加上端口号，如 192.168.159.131:9604 ，也可以修改配置文件中的remote_port变量值

/etc/ansible/hosts也可以定义一个主机范围，如192.168.159.[100:200] ，表示192.168.159.100 - 192.168.159.200 的主机

5.创建和配置 SSH 公钥认证（免密码登录）：

`ssh-keygen -t rsa`

按提示使用默认配置一路回车即可

6.通过ansible将公钥分发至各主机上：

`ansible all -m authorized_key -a "user=root key='{{ lookup('file', '/root/.ssh/id_rsa.pub')  }}' path=/root/.ssh/authorized_keys manage_dir=no" --ask-pass -c paramiko`

需要输入主机的密码，若是有的主机密码不一致，那么该主机会分发失败，此时只需再执行一遍命令输入该主机密码即可。或者先将密码相同的主机进行分组，然后依次指定主机组执行命令分批分发公钥。

此命令是通过追加的方式来推送公钥至authorized_keys，所以不用担心原来的文件内容会被覆盖。

7.修改ansible配置，指定私钥文件路径：

`sed -i "s@\#private_key_file = \/path\/to\/file@private_key_file = \/root\/.ssh\/id_rsa@g" /etc/ansible/ansible.cfg`

8.测试:

`ansible all -m ping`

可以在命令后面加上-vvvv参数查看详细的输出结果，尤其是在命令执行失败需要排错的时候非常有用。

## Reference

- [Centos7安装ansible](https://www.cnblogs.com/jackyzm/p/9578005.html)
