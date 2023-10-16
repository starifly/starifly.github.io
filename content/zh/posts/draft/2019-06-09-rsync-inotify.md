---
title: rsync和inotify配置实例
author: starifly
date: 2019-06-09T16:40:03+08:00
lastmod: 2020-03-10
categories: [rsync]
tags: [rsync,inotify]
draft: true
slug: rsync-inotify
---

Redhat(7.3)

一．安装yum

1. 检查redhat自带的yum

```shell
rpm -qa |grep yum
```

2. 卸载自带的yum

```shell
rpm -qa |grep yum | xargs rpm -e –nodeps
```

3. 检查是否完全卸载

```shell
rpm -qa |grep yum
```

4. 下载yum的相关rpm文件

```shell
curl -O http://vault.centos.org/centos/7.3.1611/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-40.el7.noarch.rpm
curl -O http://vault.centos.org//centos/7.3.1611/os/x86_64/Packages/yum-updateonboot-1.1.31-40.el7.noarch.rpm
curl -O http://vault.centos.org/centos/7.3.1611/os/x86_64/Packages/yum-utils-1.1.31-40.el7.noarch.rpm
curl -O http://vault.centos.org/centos/7.3.1611/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
curl -O http://vault.centos.org/centos/7.3.1611/os/x86_64/Packages/yum-3.4.3-150.el7.centos.noarch.rpm
```

5. 安装软件包

1）`rpm -ivh *.rpm`，这时会报错

错误：依赖检测失败：

 python-kitchen 被 yum-utils-1.1.31-40.el7.noarch 需要

下载以下：

```shell
curl -O http://mirrors.163.com/centos/7/os/x86_64/Packages/rpm-4.11.3-32.el7.x86_64.rpm
curl -O http://mirrors.163.com/centos/7/os/x86_64/Packages/rpm-libs-4.11.3-32.el7.x86_64.rpm
curl -O http://mirrors.163.com/centos/7/os/x86_64/Packages/rpm-build-libs-4.11.3-32.el7.x86_64.rpm
curl -O http://mirrors.163.com/centos/7/os/x86_64/Packages/rpm-python-4.11.3-32.el7.x86_64.rpm
curl -O http://mirrors.163.com/centos/7/os/x86_64/Packages/python-chardet-2.2.1-1.el7_1.noarch.rpm
curl -O http://mirrors.163.com/centos/7/os/x86_64/Packages/python-kitchen-1.1.1-5.el7.noarch.rpm
curl -O http://mirrors.163.com/centos/7/os/x86_64/Packages/libxml2-python-2.9.1-6.el7_2.3.x86_64.rpm
```

安装python

```shell
rpm -ivh python*

rpm -ivh libxml*
```

安装yum

```shell
rpm -ivh yum*
```

6. 配置repo文件

```shell
cd /etc/yum.repos.d

curl -o CentOS-Base.repo [http://mirrors.163.com/.help/CentOS7-Base-163.repo](http://mirrors.163.com/.help/CentOS7-Base-163.repo)
```

修改`vi CentOS-Base.repo`

```conf
[base]

baseurl=http://mirrors.163.com/centos/7/os/$basearch/

gpgkey=http://mirrors.163.com/centos/7/os/x86_64/RPM-GPG-KEY-CentOS-7

[updates]

baseurl=http://mirrors.163.com/centos/7/updates/$basearch/

gpgkey=http://mirrors.163.com/centos/7/os/x86_64/RPM-GPG-KEY-CentOS-7

[extras]

baseurl=http://mirrors.163.com/centos/7/extras/$basearch/

gpgkey=http://mirrors.163.com/centos/7/os/x86_64/RPM-GPG-KEY-CentOS-7

[centosplus]

baseurl=http://mirrors.163.com/centos/7/centosplus/$basearch/

gpgkey=http://mirrors.163.com/centos/7/os/x86_64/RPM-GPG-KEY-CentOS-7
```

7. 更新yum源

```shell
yum clean all

yum makecache
```

二．rsync

A机器：192.168.5.91

B机器：192.168.5.93

A机器：

1. 安装rsync

查看是否安装rsync

```shell
rpm -qa | grep rsync
```

如果没有安装，就执行：

```shell
yum install -y rsync
```

2. 修改配置文件

`vim /etc/rsyncd.conf`，加入以下内容：

```conf
uid = root

gid = root

hosts allow = 192.168.5.93

use chroot = no

max connections = 10

pid file = /var/run/rsyncd.pid

lock file = /var/run/rsync.lock

log file = /var/log/rsyncd.log

timeout = 600

[A]

        path = /usr/local/share

        comment = A files

        ignore errors

        read only = no

        list = no

        auth users = rsync

        secrets file = /etc/rsyncd.secrets
```

3. 创建密码文件

1）作为服务端时

`vim /etc/rsyncd.secrets`，输入以下内容

rsync:rsync_password

2）作为客户端时

`vim /root/rsyncd.secrets`，输入以下内容

rsync_password

4. 修改密码文件权限

```shell
chmod 600 /etc/rsyncd.secrets

chmod 600 /root/rsyncd.secrets
```

确认权限

```shell
ls -l /etc/rsyncd.secrets

ls -l /root/rsyncd.secrets
```

5. 启动rsync

```shell
rsync --daemon --config=/etc/rsyncd.conf
```

确认服务启动：`netstat -anp | grep 873`

6. 开机启动

```shell
echo "/usr/bin/rsync --daemon --config=/etc/rsyncd.conf" >> /etc/rc.local
```

B机器:

1. 安装rsync

查看是否安装rsync

```shell
rpm -qa | grep rsync
```

如果没有安装，就执行：

```shell
yum install -y rsync
```

2. 修改配置文件

`vim /etc/rsyncd.conf`,加入以下内容：

```conf
uid = root

gid = root

hosts allow = 192.168.5.91

use chroot = no

max connections = 10

pid file = /var/run/rsyncd.pid

lock file = /var/run/rsync.lock

log file = /var/log/rsyncd.log

timeout = 600

[B]

        path = /usr/local/share

        commetn = B files

        ignore errors

        read only = no

        list = no

        auth users = rsync

        secrets file = /etc/rsyncd.secrets
```

3. 创建密码文件

1）作为服务端时

`vim /etc/rsyncd.secrets`，输入以下内容

rsync:rsync_password

2）作为客户端时

`vim /root/rsyncd.secrets`，输入以下内容

rsync_password

4. 修改密码文件权限

```shell
chmod 600 /etc/rsyncd.secrets
```

确认权限：`ls -l /etc/rsyncd.secrets`

5. 启动rsync

```shell
rsync --daemon --config=/etc/rsyncd.conf
```

确认启动：`netstat -anp | grep 873`

6. 开机启动

```shell
echo "/usr/bin/rsync --daemon --config=/etc/rsyncd.conf" >> /etc/rc.local
```

在A机器(192.168.5.91)上测试：

把A上的 /usr/local/share/rsync 文件夹中的内容备份到B机器的 /usr/local/share 中：

```shell
rsync -vzrtopg --update --progress /usr/local/share/rsync rsync@192.168.5.93::B --password-file=/root/rsyncd.secrets
```

在B机器(192.168.5.93)上测试：

把B上的 /usr/local/share/rsync 文件夹中的内容备份到A机器的 /usr/local/share 中：

```shell
rsync -vzrtopg --update --progress /usr/local/share/rsync rsync@192.168.5.91::A --password-file=/root/rsyncd.secrets
```

加入定时任务

1. `vim rsync_backup.sh`

#!/bin/bash

2. `crontab -e`

30 8 * * * /bin/bash /path/rsync_backup.sh > /dev/null 2>&1

三．inotify

1. 安装inotify（A机器和B机器同时安装）

1）获取源代码包：

```shell
wget http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz --no-check-certificate
```

2）解压

```shell
tar -zxvf inotify-tools-3.14.tar.gz
```

3）编译安装

```shell
cd inotify-tools-3.14

mkdir /usr/local/inotify

./configure --prefix=/usr/local/inotify

make && make install
```

4）建立软连接

```shell
ls -alh /usr/local/inotify/bin/inotify*

ln -s /usr/local/inotify/bin/inotifywait /usr/bin/inotifywait

ln -s /usr/local/inotify/bin/inotifywatch /usr/bin/inotifywatch
```

2. 创建inotify监控脚本

A机器：

1）`vim rsync_inotify.sh`

```bash
#!/bin/bash

src=/usr/local/share/rsync

dest=B

user=rsync

host="192.168.5.93"

/usr/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f' -e modify,delete,create,attrib ${src} | while read file

do

        #echo "read"

        for hostip in $host

        do

                rsync -vzrtopg --update --progress $src $user@$hostip::$dest --password-file=/root/rsyncd.secrets

                echo "${file} was rsynced" >> /tmp/rsync.log 2>&1

        done

done
```

2）增加可执行权限

```shell
chmod +x rsync_inotify.sh
```

3）加入开机启动项

```shell
echo "nohup /bin/bash /root/script/rsync_inotify.sh >/dev/null 2>&1 &" >> /etc/rc.local
```

B机器：

1）`vim rsync_inotify.sh`

```bash
#!/bin/bash

src=/usr/local/share/rsync

dest=A

user=rsync

host="192.168.5.91"

/usr/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f' -e modify,delete,create,attrib ${src} | while read file

do

        echo "read"

        for hostip in $host

        do

                rsync -vzrtopg --update --progress $src $user@$hostip::$dest --password-file=/root/rsyncd.secrets

                echo "${file} was rsynced" >> /tmp/rsync.log 2>&1

        done

done
```

2）增加可执行权限

```shell
chmod +x rsync_inotify.sh
```

3）加入开机启动项

```shell
echo "nohup /bin/bash /root/script/rsync_inotify.sh >/dev/null 2>&1 &" >> /etc/rc.local
```

## Reference

- [linux rsync安装 配置 实例详解](https://m.jb51.net/article/74586.htm)
