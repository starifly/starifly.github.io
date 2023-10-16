---
title: Bash 脚本教程之 echo 命令
author: starifly
date: 2020-05-05T12:21:09+08:00
lastmod: 2020-05-05T12:21:09+08:00
categories: [shell]
tags: [shell,echo]
draft: true
slug: echo-command-of-bash-script-tutorial
---

## echo 命令

### `-n` 参数

默认情况下， `echo` 输出的文本末尾会有一个回车符。 `-n` 参数可以取消末尾的回车符，使得下一个提示符紧跟在输出内容的后面。

```bash
$ echo -n hello world
hello world$
```

上面例子中， `world` 后面直接就是下一行的提示符 `$` 。

```
$ echo a;echo b
a
b

$ echo -n a;echo b
ab
```

上面例子中， `-n` 参数可以让两个 `echo` 命令的输出连在一起，出现在同一行。

### `-e` 参数

`-e` 参数会解释引号（双引号和单引号）里面的特殊字符（比如换行符 `\n` ）。如果不使用 `-e` 参数，即默认情况下，引号会让特殊字符变成普通字符， `echo` 不解释它们，原样输出。

```bash
$ echo "Hello\nWorld"
Hello\nWorld

# 双引号的情况
$ echo -e "Hello\nWorld"
Hello
World

# 单引号的情况
$ echo -e 'Hello\nWorld'
Hello
World
```

上面代码中， `-e` 参数使得 `\n` 解释为换行符，导致输出内容里面出现换行。

## Reference

- [Bash 的基本语法 - Bash 脚本教程](https://wangdoc.com/bash/grammar.html)
