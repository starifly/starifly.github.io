---
title: ansible roles
author: starifly
date: 2023-01-26T17:17:03+08:00
lastmod: 2023-01-26T17:17:03+08:00
categories: [ansible]
tags: [ansible]
draft: true
slug: ansible-roles
---

roles：roles用于实现“**代码复用**”，roles以特定的层次型格式组织起来的playbook元素（variables, tasks, templates,handlers）；可被playbook以role的名字直接进行调用

roles的文件结构：

- files/：此角色中用到的所有文件均放置于此目录中
- templates/： Jinja2模板文件存放位置
- tasks/：任务列表文件；可以有多个，但至少有一个叫做main.yml的文件
- handlers/：处理器列表文件；可以有多个，但至少有一个叫做main.yml的文件
- vars/：变量字典文件；可以有多个，但至少有一个叫做main.yml的文件
- meta/：此角色的特殊设定及依赖关系

```
mkdir /root/roles
cd /root/roles
mkdir -p web1/{files, templayes, tasks, handlers, vars, meta}
```

```
vim web1/vars/main.yml
user: tom
group: tom
http_port: 8080
```

```
vim web1/tasks/main.yml

- name: install httpd
  yum: name=httpd state=present
- name: copy config file
  template: src=httpd.conf dest=/etc/httpd/conf/httpd.conf
  notify: restart httpd
  tags: conf
- name: start httpd
  service: name=httpd state=started enabled=true

这里的template指的是相对路径-->web1/templates
tags可以在运行时指定标签任务
```

```
vim web1/handlers/main.yml

handlers:
- name: restart httpd
  service: name=httpd state=restarted
```

```
vim web1/templates/httpd.conf

……
Listen {{ http_port }}
……
```

定义一个调用roles文件

```
vim /root/web1.yml

- hosts: web1
  remote_user: root
  roles:
    - web1
    - { role:web2, http_port:8080 }

hosts：web1 指在/etc/ansible/hosts中定义的组，上面有定义
roles: web1 指的是当前目录下的web1目录，也可通过role传递变量， 也可调用多个role
这样只需更改hosts的主机就可以实现不同主机的代码重用了
```

运行

```
ansible-playbook web1.yml
指定运行任务：
ansible-playbook -t conf web1.yml
```

roles示例，使用ansible-playbook安装zabbix

1 定义hosts

```
shell > vim /etc/ansible/hosts

[mini]

129.139.153.78:16283
155.139.190.94:12573
```

2 定义入口文件install_zabbix_agent.yml

```
shell > vim /etc/ansible/install_zabbix_agent.yml

---
- hosts: mini
  roles:
  - install_zabbix_agent

## 可以看到将要安装的主机组为 mini 组，角色为 install_zabbix_agent
```

3 定义角色 install_zabbix_agent

```
shell > tree /etc/ansible/roles/install_zabbix_agent/

├── files
│    └── zabbix-2.4.5.tar.gz
├── tasks
│    └── main.yml
├── templates
│    ├── zabbix_agentd
│    └── zabbix_agentd.conf
└── vars
      └── main.yml

## 建立 files     目录，存放编译安装过的 zabbix_agent 目录的压缩文件，用于拷贝到远程主机
## 建立 tasks     目录，用于编写将要执行的任务
## 建立 templates 目录，用于存放可变的模板文件
## 建立 vars      目录，用于存放变量信息
```

4 建立tasks主文件

```
shell > cat /etc/ansible/roles/install_zabbix_agent/tasks/main.yml

---
  - name: Install Software
    yum: name={{ item }} state=latest
    with_items:
      - libcurl-devel
  - name: Create Zabbix User
    user: name={{ zabbix_user }} state=present createhome=no shell=/sbin/nologin
  - name: Copy Zabbix.tar.gz
    copy: src=zabbix-{{ zabbix_version }}.tar.gz dest={{ zabbix_dir }}/src/zabbix-{{ zabbix_version }}.tar.gz owner=root group=root
  - name: Uncompression Zabbix.tar.gz
    shell: tar zxf {{ zabbix_dir }}/src/zabbix-{{ zabbix_version }}.tar.gz -C {{ zabbix_dir }}/
  - name: Copy Zabbix Start Script
    template: src=zabbix_agentd dest=/etc/init.d/zabbix_agentd owner=root group=root mode=0755
  - name: Copy Zabbix Config File
    template: src=zabbix_agentd.conf dest={{ zabbix_dir }}/zabbix/etc/zabbix_agentd.conf owner={{ zabbix_user }} group={{ zabbix_user }} mode=0644
  - name: Modify Zabbix Dir Permisson
    file: path={{ zabbix_dir }}/zabbix owner={{ zabbix_user }} group={{ zabbix_user }} mode=0755 recurse=yes
  - name: Start Zabbix Service
    shell: /etc/init.d/zabbix_agentd start
  - name: Add Boot Start Zabbix Service
    shell: chkconfig --level 35 zabbix_agentd on
```

5 建立主变量文件

```
shell > cat /etc/ansible/roles/install_zabbix_agent/vars/main.yml

zabbix_dir: /usr/local
zabbix_version: 2.4.5
zabbix_user: zabbix
zabbix_port: 10050
zabbix_server_ip: 131.142.101.120
```

6 建立模板文件

```
shell > cat /etc/ansible/roles/install_zabbix_agent/templates/zabbix_agentd

#!/bin/bash
#
# chkconfig: - 90 10
# description:  Starts and stops Zabbix Agent using chkconfig
#                               Tested on Fedora Core 2 - 5
#                               Should work on all Fedora Core versions
#
# @name:        zabbix_agentd
# @author:      Alexander Hagenah <hagenah@topconcepts.com>
# @created:     18.04.2006
#
# Modified for Zabbix 2.0.0
# May 2012, Zabbix SIA
#
# Source function library.
. /etc/init.d/functions

# Variables
# Edit these to match your system settings

        # Zabbix-Directory
        BASEDIR={{ zabbix_dir }}/zabbix

        # Binary File
        BINARY_NAME=zabbix_agentd

        # Full Binary File Call
        FULLPATH=$BASEDIR/sbin/$BINARY_NAME

        # PID file
        PIDFILE=/tmp/$BINARY_NAME.pid

        # Establish args
        ERROR=0
        STOPPING=0

#
# No need to edit the things below
#

# application checking status
if [ -f $PIDFILE  ] && [ -s $PIDFILE ]
        then
        PID=`cat $PIDFILE`

        if [ "x$PID" != "x" ] && kill -0 $PID 2>/dev/null && [ $BINARY_NAME == `ps -e | grep $PID | awk '{print $4}'` ]
        then
                STATUS="$BINARY_NAME (pid `pidof $APP`) running.."
                RUNNING=1
        else
                rm -f $PIDFILE
                STATUS="$BINARY_NAME (pid file existed ($PID) and now removed) not running.."
                RUNNING=0
        fi
else
        if [ `ps -e | grep $BINARY_NAME | head -1 | awk '{ print $1 }'` ]
                then
                STATUS="$BINARY_NAME (pid `pidof $APP`, but no pid file) running.."
        else
                STATUS="$BINARY_NAME (no pid file) not running"
        fi
        RUNNING=0
fi

# functions
start() {
        if [ $RUNNING -eq 1 ]
                then
                echo "$0 $ARG: $BINARY_NAME (pid $PID) already running"
        else
                action $"Starting $BINARY_NAME: " $FULLPATH
                touch /var/lock/subsys/$BINARY_NAME
        fi
}

stop() {
        echo -n $"Shutting down $BINARY_NAME: "
        killproc $BINARY_NAME
        RETVAL=$?
        echo
        [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/$BINARY_NAME
        RUNNING=0
}


# logic
case "$1" in
        start)
                start
                ;;
        stop)
                stop
                ;;
        status)
                status $BINARY_NAME
                ;;
        restart)
                stop
                sleep 10
                start
                ;;
        help|*)
                echo $"Usage: $0 {start|stop|status|restart|help}"
                cat <<EOF

                        start           - start $BINARY_NAME
                        stop            - stop $BINARY_NAME
                        status          - show current status of $BINARY_NAME
                        restart         - restart $BINARY_NAME if running by sending a SIGHUP or start if not running
                        help            - this screen

EOF
        exit 1
        ;;
esac

exit 0
```

```
shell > cat /etc/ansible/roles/install_zabbix_agent/templates/zabbix_agentd.conf

# This is a config file for the Zabbix agent daemon (Unix)
# To get more information about Zabbix, visit http://www.zabbix.com

############ GENERAL PARAMETERS #################

### Option: PidFile
#       Name of PID file.
#
# Mandatory: no
# Default:
# PidFile=/tmp/zabbix_agentd.pid

### Option: LogFile
#       Name of log file.
#       If not set, syslog is used.
#
# Mandatory: no
# Default:
# LogFile=

LogFile=/tmp/zabbix_agentd.log

### Option: LogFileSize
#       Maximum size of log file in MB.
#       0 - disable automatic log rotation.
#
# Mandatory: no
# Range: 0-1024
# Default:
# LogFileSize=1

### Option: DebugLevel
#       Specifies debug level
#       0 - basic information about starting and stopping of Zabbix processes
#       1 - critical information
#       2 - error information
#       3 - warnings
#       4 - for debugging (produces lots of information)
#
# Mandatory: no
# Range: 0-4
# Default:
# DebugLevel=3

### Option: SourceIP
#       Source IP address for outgoing connections.
#
# Mandatory: no
# Default:
# SourceIP=

### Option: EnableRemoteCommands
#       Whether remote commands from Zabbix server are allowed.
#       0 - not allowed
#       1 - allowed
#
# Mandatory: no
# Default:
# EnableRemoteCommands=0

### Option: LogRemoteCommands
#       Enable logging of executed shell commands as warnings.
#       0 - disabled
#       1 - enabled
#
# Mandatory: no
# Default:
# LogRemoteCommands=0

##### Passive checks related

### Option: Server
#       List of comma delimited IP addresses (or hostnames) of Zabbix servers.
#       Incoming connections will be accepted only from the hosts listed here.
#       If IPv6 support is enabled then '127.0.0.1', '::127.0.0.1', '::ffff:127.0.0.1' are treated equally.
#
# Mandatory: no
# Default:
# Server=

Server={{ zabbix_server_ip }}

### Option: ListenPort
#       Agent will listen on this port for connections from the server.
#
# Mandatory: no
# Range: 1024-32767
# Default:
# ListenPort=10050
ListenPort={{ zabbix_port }}

### Option: ListenIP
#       List of comma delimited IP addresses that the agent should listen on.
#       First IP address is sent to Zabbix server if connecting to it to retrieve list of active checks.
#
# Mandatory: no
# Default:
# ListenIP=0.0.0.0

### Option: StartAgents
#       Number of pre-forked instances of zabbix_agentd that process passive checks.
#       If set to 0, disables passive checks and the agent will not listen on any TCP port.
#
# Mandatory: no
# Range: 0-100
# Default:
# StartAgents=3

##### Active checks related

### Option: ServerActive
#       List of comma delimited IP:port (or hostname:port) pairs of Zabbix servers for active checks.
#       If port is not specified, default port is used.
#       IPv6 addresses must be enclosed in square brackets if port for that host is specified.
#       If port is not specified, square brackets for IPv6 addresses are optional.
#       If this parameter is not specified, active checks are disabled.
#       Example: ServerActive=127.0.0.1:20051,zabbix.domain,[::1]:30051,::1,[12fc::1]
#
# Mandatory: no
# Default:
# ServerActive=

#ServerActive=127.0.0.1:10051

### Option: Hostname
#       Unique, case sensitive hostname.
#       Required for active checks and must match hostname as configured on the server.
#       Value is acquired from HostnameItem if undefined.
#
# Mandatory: no
# Default:
# Hostname=

Hostname={{ ansible_all_ipv4_addresses[1] }}

### Option: HostnameItem
#       Item used for generating Hostname if it is undefined. Ignored if Hostname is defined.
#       Does not support UserParameters or aliases.
#
# Mandatory: no
# Default:
# HostnameItem=system.hostname

### Option: HostMetadata
#       Optional parameter that defines host metadata.
#       Host metadata is used at host auto-registration process.
#       An agent will issue an error and not start if the value is over limit of 255 characters.
#       If not defined, value will be acquired from HostMetadataItem.
#
# Mandatory: no
# Range: 0-255 characters
# Default:
# HostMetadata=

### Option: HostMetadataItem
#       Optional parameter that defines an item used for getting host metadata.
#       Host metadata is used at host auto-registration process.
#       During an auto-registration request an agent will log a warning message if
#       the value returned by specified item is over limit of 255 characters.
#       This option is only used when HostMetadata is not defined.
#
# Mandatory: no
# Default:
# HostMetadataItem=

### Option: RefreshActiveChecks
#       How often list of active checks is refreshed, in seconds.
#
# Mandatory: no
# Range: 60-3600
# Default:
# RefreshActiveChecks=120

### Option: BufferSend
#       Do not keep data longer than N seconds in buffer.
#
# Mandatory: no
# Range: 1-3600
# Default:
# BufferSend=5

### Option: BufferSize
#       Maximum number of values in a memory buffer. The agent will send
#       all collected data to Zabbix Server or Proxy if the buffer is full.
#
# Mandatory: no
# Range: 2-65535
# Default:
# BufferSize=100

### Option: MaxLinesPerSecond
#       Maximum number of new lines the agent will send per second to Zabbix Server
#       or Proxy processing 'log' and 'logrt' active checks.
#       The provided value will be overridden by the parameter 'maxlines',
#       provided in 'log' or 'logrt' item keys.
#
# Mandatory: no
# Range: 1-1000
# Default:
# MaxLinesPerSecond=100

############ ADVANCED PARAMETERS #################

### Option: Alias
#       Sets an alias for an item key. It can be used to substitute long and complex item key with a smaller and simpler one.
#       Multiple Alias parameters may be present. Multiple parameters with the same Alias key are not allowed.
#       Different Alias keys may reference the same item key.
#       For example, to retrieve the ID of user 'zabbix':
#       Alias=zabbix.userid:vfs.file.regexp[/etc/passwd,^zabbix:.:([0-9]+),,,,\1]
#       Now shorthand key zabbix.userid may be used to retrieve data.
#       Aliases can be used in HostMetadataItem but not in HostnameItem parameters.
#
# Mandatory: no
# Range:
# Default:

### Option: Timeout
#       Spend no more than Timeout seconds on processing
#
# Mandatory: no
# Range: 1-30
# Default:
Timeout=20

### Option: AllowRoot
#       Allow the agent to run as 'root'. If disabled and the agent is started by 'root', the agent
#       will try to switch to the user specified by the User configuration option instead.
#       Has no effect if started under a regular user.
#       0 - do not allow
#       1 - allow
#
# Mandatory: no
# Default:
# AllowRoot=0

### Option: User
#       Drop privileges to a specific, existing user on the system.
#       Only has effect if run as 'root' and AllowRoot is disabled.
#
# Mandatory: no
# Default:
# User=zabbix

### Option: Include
#       You may include individual files or all files in a directory in the configuration file.
#       Installing Zabbix will create include directory in /usr/local/etc, unless modified during the compile time.
#
# Mandatory: no
# Default:
# Include=

# Include=/usr/local/etc/zabbix_agentd.userparams.conf
# Include=/usr/local/etc/zabbix_agentd.conf.d/
# Include=/usr/local/etc/zabbix_agentd.conf.d/*.conf

####### USER-DEFINED MONITORED PARAMETERS #######

### Option: UnsafeUserParameters
#       Allow all characters to be passed in arguments to user-defined parameters.
#       0 - do not allow
#       1 - allow
#
# Mandatory: no
# Range: 0-1
# Default:
UnsafeUserParameters=1

### Option: UserParameter
#       User-defined parameter to monitor. There can be several user-defined parameters.
#       Format: UserParameter=<key>,<shell command>
#       See 'zabbix_agentd' directory for examples.
#
# Mandatory: no
# Default:
# UserParameter=

####### LOADABLE MODULES #######

### Option: LoadModulePath
#       Full path to location of agent modules.
#       Default depends on compilation options.
#
# Mandatory: no
# Default:
# LoadModulePath=${libdir}/modules

### Option: LoadModule
#       Module to load at agent startup. Modules are used to extend functionality of the agent.
#       Format: LoadModule=<module.so>
#       The modules must be located in directory specified by LoadModulePath.
#       It is allowed to include multiple LoadModule parameters.
#
# Mandatory: no
# Default:
# LoadModule=
```

7 安装

```
shell > ansible-playbook /etc/ansible/install_zabbix_agent.yml

PLAY [mini] *******************************************************************

GATHERING FACTS ***************************************************************
ok: [129.139.153.78]
ok: [155.139.190.94]

TASK: [install_zabbix_agent | Install Software] *******************************
changed: [155.139.190.94] => (item=libcurl-devel)
changed: [129.139.153.78] => (item=libcurl-devel)

TASK: [install_zabbix_agent | Create Zabbix User] *****************************
changed: [129.139.153.78]
changed: [155.139.190.94]

TASK: [install_zabbix_agent | Copy Zabbix.tar.gz] *****************************
changed: [129.139.153.78]
changed: [155.139.190.94]

TASK: [install_zabbix_agent | Uncompression Zabbix.tar.gz] ********************
changed: [129.139.153.78]
changed: [155.139.190.94]

TASK: [install_zabbix_agent | Copy Zabbix Start Script] ***********************
changed: [155.139.190.94]
changed: [129.139.153.78]

TASK: [install_zabbix_agent | Copy Zabbix Config File] ************************
changed: [129.139.153.78]
changed: [155.139.190.94]

TASK: [install_zabbix_agent | Modify Zabbix Dir Permisson] ********************
changed: [155.139.190.94]
changed: [129.139.153.78]

TASK: [install_zabbix_agent | Start Zabbix Service] ***************************
changed: [129.139.153.78]
changed: [155.139.190.94]

TASK: [install_zabbix_agent | Add Boot Start Zabbix Service] ******************
changed: [129.139.153.78]
changed: [155.139.190.94]

PLAY RECAP ********************************************************************
155.139.190.94               : ok=10   changed=9    unreachable=0    failed=0
129.139.153.78               : ok=10   changed=9    unreachable=0    failed=0

## 关注一下，启动脚本跟配置文件中变量的引用。
## 完成安装，可以去客户机检查效果了 ！
```

## Reference

- [ansible自动化运维详细教程及playbook详解](https://blog.csdn.net/zhao12795969/article/details/80888492)
