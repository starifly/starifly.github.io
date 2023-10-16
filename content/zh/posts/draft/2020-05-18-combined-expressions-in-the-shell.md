---
title: shell中的结合表达式
author: starifly
date: 2020-05-18T21:53:50+08:00
lastmod: 2020-05-18T21:53:50+08:00
categories: [shell]
tags: [shell]
draft: true
slug: combined-expressions-in-the-shell
---

有三个用于`test`和`[[  ]]`的逻辑操作，它们是`AND`、`OR`和`NOT`。`test`和`[[  ]]`使用不通的操作符来表示这些操作：


操作符 | test | [[  ]]
---- | ---- | ----
AND | -a | &&
OR | -o | \|\|
NOT | ! | !

实例：

`if [[ INT -ge MIN_VAL && INT -le MAX_VAL ]]; then`

或者

`if [ $INT -ge $MIN_VAL -a $INT -le $MAX_VAL ]; then`

- [TLCL](http://billie66.github.io/TLCL/book/)
