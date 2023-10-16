---
title: linux普通用户下如何执行需要 root 权限的程序
author: starifly
date: 2020-01-21T22:46:40+08:00
categories: [linux]
tags: [linux]
draft: true
slug: how-to-execute-programs-that-require-root-privileges-under-ordinary-linux-user
---

如何让普通用户执行需要 root 权限的程序，这里提供两种方法，一是使用 sudo，二是利用 chmod 配置特殊权限。

## 安全的让普通用户执行 sudo

1. 对 sudo 里的命令，尽量明确给出参数，比如上面的例子我们可以改为
```conf
   Defaults:admin !requiretty
   admin ALL=(ALL) NOPASSWD: /nginx/sbin/nginx -t
   admin ALL=(ALL) NOPASSWD: /nginx/sbin/nginx -s reload
```
   这样普通用户执行的时候只有加上给出的这些参数，而不是任何其他内容  
2. 对 sudo 里面的命令，都加上 chattr 属性，比如 `chattr +i /nginx/sbin/nginx`  
   这样普通用户将无法修改/nginx/sbin/nginx 也无法执行 `mv /nginx/sbin/nginx /tmp/`  
3. 对 /etc/sudoers 加上 chattr，比如 `chattr +i /etc/sudoers` 这样普通用户将无法修改/etc/sudoers

## chmod 配置特殊权限

首先在 root 用户 执行 `chmod u+s /nginx/sbin/nginx` 给予 suid 权限，这样程序允许普通用户以root身份暂时执行该程序，并在执行结束后再恢复身份。

这就是 s 的功能，但是在整个系统中特别是 root 本身，最好不要过多的设置这种类型的文件（除非必要）这样可以保证系统的安全，避免因为某些程序的 bug 而是系统遭到入侵。
