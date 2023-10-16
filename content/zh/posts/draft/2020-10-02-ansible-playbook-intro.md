---
title: Ansible之Playbook
author: starifly
date: 2020-10-02T17:46:44+08:00
lastmod: 2020-10-02T17:46:44+08:00
categories: [ansible]
tags: [ansible,playbook]
draft: true
slug: ansible-playbook-intro
---

## Playbook介绍

`Playbook` 与 `ad-hoc` 相比,是一种完全不同的运用ansible的方式，类似与 `saltstack` 的 `state` 状态文件。`ad-hoc` 无法持久使用，`playbook` 可以持久使用。

`playbook` 是由一个或多个 `play` 组成的列表，`play` 的主要功能在于将事先归并为一组的主机装扮成事先通过 `ansible` 中的 `task` 定义好的角色。从根本上来讲，所谓的 `task` 无非是调用 `ansible` 的一个 `module`。将多个 `play` 组织在一个 `playbook` 中，即可以让它们联合起来按事先编排的机制完成某一任务。

## Playbook核心元素

- Hosts 执行的远程主机列表
- Tasks 任务集
- Varniables 内置变量或自定义变量在playbook中调用
- Templates 模板，即使用模板语法的文件，比如配置文件等
- Handlers 和notity结合使用，由特定条件触发的操作，满足条件方才执行，否则不执行
- tags 标签，指定某条任务执行，用于选择运行playbook中的部分代码。

## Playbook语法

`playbook` 使用 `yaml` 语法格式，后缀可以是 `yaml`,也可以是 `yml`。

- 在单一一个 `playbook` 文件中，可以连续三个连子号(`---`)区分多个 `play`。还有选择性的连续三个点好(`...`)用来表示 `play` 的结尾，也可省略。
- 次行开始正常写 `playbook` 的内容，一般都会写上描述该 `playbook` 的功能。
- 使用#号注释代码。
- 缩进必须统一，不能空格和 `tab` 混用。
- 缩进的级别也必须是一致的，同样的缩进代表同样的级别，程序判别配置的级别是通过缩进结合换行实现的。
- `YAML` 文件内容和 `Linux` 系统大小写判断方式保持一致，是区分大小写的，`k/v` 的值均需大小写敏感
- `k/v` 的值可同行写也可以换行写。同行使用:分隔。
- `v` 可以是个字符串，也可以是一个列表
- 一个完整的代码块功能需要最少元素包括 `name: task`

## Playbook中元素属性

### 主机与用户

在一个 `playbook` 开始时，最先定义的是要操作的主机和用户

```yaml
---
- hosts: 192.168.1.31
  remote_user: root
```

除了上面的定义外，还可以在某一个 `tasks` 中定义要执行该任务的远程用户

```yaml
tasks: 
  - name: run df -h
    remote_user: test
    shell: name=df -h
```

还可以定义使用 `sudo` 授权用户执行该任务

```yaml
tasks: 
  - name: run df -h
    sudo_user: test
    sudo: yes
    shell: name=df -h
```

### tasks任务列表

每一个 `task` 必须有一个名称 `name`,这样在运行 `playbook` 时，从其输出的任务执行信息中可以很清楚的辨别是属于哪一个 `task` 的，如果没有定义 `name`，`action` 的值将会用作输出信息中标记特定的 `task`。

每一个 `playbook` 中可以包含一个或者多个 `tasks` 任务列表，每一个 `tasks` 完成具体的一件事，（任务模块）比如创建一个用户或者安装一个软件等，在 `hosts` 中定义的主机或者主机组都将会执行这个被定义的 `tasks`。

```yaml
tasks:
  - name: create new file
    file: path=/tmp/test01.txt state=touch
  - name: create new user
    user: name=test001 state=present
```

### Handlers与Notify

很多时候当我们某一个配置发生改变，我们需要重启服务，（比如 `httpd` 配置文件文件发生改变了）这时候就可以用到 `handlers` 和 `notify` 了；(当发生改动时) `notify actions` 会在 `playbook` 的每一个 `task` 结束时被触发，而且即使有多个不同 `task` 通知改动的发生，`notify actions` 只会被触发一次；比如多个 `resources` 指出因为一个配置文件被改动，所以 `apache` 需要重启，但是重新启动的操作只会被执行一次。

```yaml
[root@ansible ~]# cat httpd.yml 
#用于安装httpd并配置启动
---
- hosts: 192.168.1.31
  remote_user: root

  tasks:
  - name: install httpd
    yum: name=httpd state=installed
  - name: config httpd
    template: src=/root/httpd.conf dest=/etc/httpd/conf/httpd.conf
    notify:
      - restart httpd
  - name: start httpd
    service: name=httpd state=started

  handlers:
    - name: restart httpd
      service: name=httpd state=restarted

#这里只要对httpd.conf配置文件作出了修改，修改后需要重启生效，在tasks中定义了restart httpd这个action，然后在handlers中引用上面tasks中定义的notify。
```

## Playbook中变量的使用

环境说明：这里配置了两个组，一个apache组和一个nginx组

```
[root@ansible PlayBook]# cat /etc/ansible/hosts
[apache]
192.168.1.36
192.168.1.33

[nginx]
192.168.1.3[1:2]
```

### 命令行指定变量

执行 `playbook` 时候通过参数 `-e` 传入变量，这样传入的变量在整个 `playbook` 中都可以被调用，属于全局变量

```yaml
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root

  tasks:
    - name: install pkg
      yum: name={{ pkg }}

#执行playbook 指定pkg
[root@ansible PlayBook]# ansible-playbook -e "pkg=httpd" variables.yml
```

### hosts文件中定义变量

在 `/etc/ansible/hosts` 文件中定义变量，可以针对每个主机定义不同的变量，也可以定义一个组的变量，然后直接在 `playbook` 中直接调用。注意，组中定义的变量没有单个主机中的优先级高。

```yaml
# 编辑hosts文件定义变量
[root@ansible PlayBook]# vim /etc/ansible/hosts
[apache]
192.168.1.36 webdir=/opt/test     #定义单个主机的变量
192.168.1.33
[apache:vars]      #定义整个组的统一变量
webdir=/web/test

[nginx]
192.168.1.3[1:2]
[nginx:vars]
webdir=/opt/web


# 编辑playbook文件
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root

  tasks:
    - name: create webdir
      file: name={{ webdir }} state=directory   #引用变量


# 执行playbook
[root@ansible PlayBook]# ansible-playbook variables.yml
```

### playbook文件中定义变量

编写 `playbook` 时，直接在里面定义变量，然后直接引用，可以定义多个变量；注意：如果在执行 `playbook` 时，又通过 `-e` 参数指定变量的值，那么会以 `-e` 参数指定的为准。

```yaml
# 编辑playbook
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root
  vars:                #定义变量
    pkg: nginx         #变量1
    dir: /tmp/test1    #变量2

  tasks:
    - name: install pkg
      yum: name={{ pkg }} state=installed    #引用变量
    - name: create new dir
      file: name={{ dir }} state=directory   #引用变量


# 执行playbook
[root@ansible PlayBook]# ansible-playbook variables.yml

# 如果执行时候又重新指定了变量的值，那么会已重新指定的为准
[root@ansible PlayBook]# ansible-playbook -e "dir=/tmp/test2" variables.yml
```

### 调用setup模块获取变量

`setup` 模块默认是获取主机信息的，有时候在 `playbook` 中需要用到，所以可以直接调用。常用的参数[参考](https://buji595.github.io/2019/05/27/Ansible%20Ad-hoc%E5%B8%B8%E7%94%A8Module/#setup)

```yaml
# 编辑playbook文件
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root

  tasks:
    - name: create file
      file: name={{ ansible_fqdn }}.log state=touch   #引用setup中的ansible_fqdn


# 执行playbook
[root@ansible PlayBook]# ansible-playbook variables.yml
```

### 独立的变量YAML文件中定义

为了方便管理将所有的变量统一放在一个独立的变量 `YAML` 文件中，`laybook` 文件直接引用文件调用变量即可。

```yaml
# 定义存放变量的文件
[root@ansible PlayBook]# cat var.yml 
var1: vsftpd
var2: httpd

# 编写playbook
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root
  vars_files:    #引用变量文件
    - ./var.yml   #指定变量文件的path（这里可以是绝对路径，也可以是相对路径）

  tasks:
    - name: install package
      yum: name={{ var1 }}   #引用变量
    - name: create file
      file: name=/tmp/{{ var2 }}.log state=touch   #引用变量


# 执行playbook
[root@ansible PlayBook]# ansible-playbook  variables.yml
```

## Playbook中模板的使用

`template` 模板为我们提供了动态配置服务，使用 `jinja2` 语言，里面支持多种条件判断、循环、逻辑运算、比较操作等。其实说白了也就是一个文件，和之前配置文件使用 `copy` 一样，只是使用 `copy`，不能根据服务器配置不一样进行不同动态的配置。这样就不利于管理。

说明：

1、多数情况下都将 `template` 文件放在和 `playbook` 文件同级的 `templates` 目录下（手动创建），这样 `playbook` 文件中可以直接引用，会自动去找这个文件。如果放在别的地方，也可以通过绝对路径去指定。
2、模板文件后缀名为 `.j2`。

## Reference

- [Ansible--Ansible之Playbook](https://www.cnblogs.com/yanjieli/p/10969299.html)
