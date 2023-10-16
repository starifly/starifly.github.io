---
title: Linux中执行shell脚本的方式
author: starifly
date: 2020-05-11T22:41:47+08:00
lastmod: 2020-05-11T22:41:47+08:00
categories: [linux,shell]
tags: [linux,shell]
draft: true
slug: how-to-execute-shell-script-in-linux
---

用`source`和点命令`.`执行脚本时不改变当前进程，并且脚本中的变量可以看到。该脚本文件可以无“执行权限”。

而直接执行脚本文件和使用`sh`执行脚本时改变了当前进程，并且脚本中的变量不能看到。其中使用`sh`时该脚本文件可以无”执行权限“，因为此时脚本是作为参数传给`sh`命令来执行的，不是脚本自己来执行。

## Reference

- [Linux 中执行Shell 脚本的方式（三种方法）](https://blog.csdn.net/timchen525/article/details/76407735)
