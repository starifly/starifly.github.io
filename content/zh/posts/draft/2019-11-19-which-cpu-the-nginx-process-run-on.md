---
title: 观察nginx进程运行在哪颗cpu上
author: starifly
date: 2019-11-19T22:43:18+08:00
categories: [nginx]
tags: [nginx,linux]
draft: true
slug: which-cpu-the-nginx-process-run-on
---

`ps axo comm,pid,psr | grep nginx`

动态观测

`watch -n.5 'ps axo comm,pid,psr | grep nginx'`
