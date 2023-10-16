---
title: git add -A 和 git add . 的区别
author: starifly
date: 2018-08-23T20:18:10+08:00
categories: [git]
tags: [git]
slug: git-add-difference
---

git add -A、git add .和git add -u在功能上看似相近，但是有细微的差别，而且功能会随着git版本的不同而不同。

<!--more-->

## 区别

Git Version 1.x: 

   ![](/images/git-add-difference-01.jpg "Git Version 1.x")

Git Version 2.x: 

   ![](/images/git-add-difference-02.jpg "Git Version 2.x")

## 总结
- git add -A就不用介绍了，它在任何情况下都是包括所有的变动。
- git add .在git版本是2.x的时候等同于git add -A，而当版本是1.x的时候**不包括被删除的文件**。
- git add -u仅监控已经被add的文件（即tracked file），**不会提交新文件（untracked file）**。

希望本文对你有所帮助。
