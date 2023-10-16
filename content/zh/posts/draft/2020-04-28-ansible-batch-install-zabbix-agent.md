---
title: Ansible批量安装zabbix-agent
author: starifly
date: 2020-04-28T21:09:30+08:00
lastmod: 2020-07-04
categories: [ansible]
tags: [ansible,zabbix]
draft: true
slug: ansible-batch-install-zabbix-agent
---

## 添加主机清单

`vim /etc/ansible/hosts`

```conf
192.168.5.136   name=192.168.5.136   zabbix_server=192.168.5.159 ansible_ssh_port=6422 ansible_ssh_pass=
```

## 新增playbook

```yaml
- hosts: all
  remote_user: root
  tasks:
  - name:  RedHat6 64 family copy zabbix-agent rpm
    copy: src=/tmp/zabbix-agent-4.4.6-1.el7.x86_64.rpm dest=/tmp/zabbix-agent-4.4.6-1.el7.x86_64.rpm
    when: 
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "6"
      - ansible_machine == "x86_64" 
  - name:  RedHat7 64 family copy zabbix-agent rpm
    copy: src=/tmp/zabbix-agent-4.4.6-1.el7.x86_64.rpm dest=/tmp/zabbix-agent-4.4.6-1.el7.x86_64.rpm
    when: 
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "7"
      - ansible_machine == "x86_64" 
  - name:  RedHat8 64 family copy zabbix-agent rpm
    copy: src=/tmp/zabbix-agent-4.4.6-1.el8.x86_64.rpm dest=/tmp/zabbix-agent-4.4.6-1.el8.x86_64.rpm
    when: 
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "8"
      - ansible_machine == "x86_64" 
  - name:  RedHat6 32 family copy zabbix-agent rpm
    copy: src=/tmp/zabbix-agent-4.4.6-1.el6.i686.rpm dest=/tmp/zabbix-agent-4.4.6-1.el6.i686.rpm
    when:
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "6"
      - ansible_machine == "i686" 
  - name:  RedHat6 family copy zabbix script
    copy: src=/tmp/zabbix_script.tar.gz dest=/tmp/zabbix_script.tar.gz
    when:
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "6" 
  - name:  RedHat7 family copy zabbix script
    copy: src=/tmp/zabbix_script.tar.gz dest=/tmp/zabbix_script.tar.gz
    when:
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "7" 
  - name:  RedHat8 family copy zabbix script
    copy: src=/tmp/zabbix_script.tar.gz dest=/tmp/zabbix_script.tar.gz
    when:
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "8" 
  - name: Shutdown selinux
    shell: setenforce 0;sed -i s#SELINUX=enforcing#SElINUX=disabled#g /etc/selinux/config
  - name: yum install iostat
    shell: rpm -qa | grep sysstat || yum install -y sysstat
  - name: yum install zabbix-agent
    shell: cd /tmp;yum -y localinstall zabbix-agent-4.4.*.rpm
  - name: rm zabbix-agent
    shell: cd /tmp;rm -rf zabbix-agent-4.*.rpm
  - name: configure Server IP
    shell: sed -i 's/Server=127.0.0.1/Server={{zabbix_server}}/' /etc/zabbix/zabbix_agentd.conf  
  - name: configure ServerActive IP
    shell: sed -i 's/ServerActive=127.0.0.1/ServerActive={{zabbix_server}}/' /etc/zabbix/zabbix_agentd.conf
  - name: configure HostMetadataItem
    shell: sed -i 's/# HostMetadataItem=/HostMetadataItem=system.uname/' /etc/zabbix/zabbix_agentd.conf
  - name: add scripts
    shell: cd /tmp;tar -zxf zabbix_script.tar.gz -C /etc/zabbix/zabbix_agentd.d/
  - name: rm zabbix script
    shell: cd /tmp;rm -rf zabbix_script.tar.gz
  - name: Configure Hostname
    shell: sed -i 's/Hostname=Zabbix server/Hostname='{{ansible_default_ipv4.address}}'/' /etc/zabbix/zabbix_agentd.conf
  - name: Redhat6 family start zabbix
    shell: /etc/init.d/zabbix-agent start;chkconfig zabbix-agent on
    when: 
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "6"
  - name: Redhat7 family start zabbix
    shell: systemctl enable zabbix-agent.service;systemctl restart zabbix-agent.service
    when: 
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "7"
  - name: Redhat8 family start zabbix
    shell: systemctl enable zabbix-agent.service;systemctl restart zabbix-agent.service
    when: 
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "8"
```

执行 `ansible-playbook zabbix_agent.yaml` 即可开始批量安装。

## Reference

- [ansible 批量安装 Centos7&Centos6 zabbix-agent客户端](https://blog.csdn.net/xiegh2014/article/details/80758603)
- [Zabbix agent 批量分发部署](https://www.jianshu.com/p/cd49b9ddf92f)
