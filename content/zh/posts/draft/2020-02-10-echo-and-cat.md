---
title: 使用echo和cat时一些注意的点
author: starifly
date: 2020-02-10T21:08:16+08:00
lastmod: 2020-05-05
categories: [shell]
tags: [shell,linux,cat,echo]
draft: true
slug: echo-and-cat
---

- `cat` 多文本输入时，末尾的 EOF 标记要顶格写
- `echo` 接双引号会解析其中的变量
- `cat > file <<EOF` 会解析其中的变量，而 `cat > file <<'EOF'`不会
- 如果文本包含就要解析的变量又包含不要解析的变量，要用 `echo` 加单引号的方式，单引号中要解析的变量再加单引号
