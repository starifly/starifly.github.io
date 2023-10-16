---
title: Bash 脚本教程之变量
author: starifly
date: 2020-05-05T21:03:10+08:00
lastmod: 2020-05-05T21:03:10+08:00
categories: [shell]
tags: [shell]
draft: true
slug: variable-of-bash-script-tutorial
---

Bash 变量分成环境变量和自定义变量两类。

注意，Bash 变量名区分大小写，`HOME`和`home`是两个不同的变量。

## 创建变量

Bash 没有数据类型的概念，所有的变量值都是字符串。

下面是一些自定义变量的例子。

```
a=z                     # 变量 a 赋值为字符串 z
b="a string"            # 变量值包含空格，就必须放在引号里面
c="a string and $b"     # 变量值可以引用其他变量的值
d="\t\ta string\n"      # 变量值可以使用转义字符
e=$(ls -l foo.txt)      # 变量值可以是命令的执行结果
f=$((5 * 7))            # 变量值可以是数学运算的结果
```

## 读取变量

读取变量的时候，直接在变量名前加上`$`就可以了。

如果变量的值本身也是变量，可以使用`${!varname}`的语法，读取最终的值。

```
$ myvar=USER
$ echo ${!myvar}
ruanyf
```

上面的例子中，变量`myvar`的值是`USER`，`${!myvar}`的写法将其展开成最终的值。

## 输出变量，export 命令

用户创建的变量仅可用于当前 Shell，子 Shell 默认读取不到父 Shell 定义的变量。为了把变量传递给子 Shell，需要使用`export`命令。这样输出的变量，对于子 Shell 来说就是环境变量。

`export`命令用来向子 Shell 输出变量。

```
export NAME=value
```

上面命令执行后，当前 Shell 及随后新建的子 Shell，都可以读取变量`$NAME`。

子 Shell 如果修改继承的变量，不会影响父 Shell。

```
# 输出变量 $foo
$ export foo=bar

# 新建子 Shell
$ bash

# 读取 $foo
$ echo $foo
bar

# 修改继承的变量
$ foo=baz

# 退出子 Shell
$ exit

# 读取 $foo
$ echo $foo
bar
```

上面例子中，子 Shell 修改了继承的变量`$foo`，对父 Shell 没有影响。

## 变量的默认值

Bash 提供四个特殊语法，跟变量的默认值有关，目的是保证变量不为空。

```
${varname:-word}
```

上面语法的含义是，如果变量`varname`存在且不为空，则返回它的值，否则返回`word`。它的目的是返回一个默认值，比如`${count:-0}`表示变量`count`不存在时返回`0`。

```
${varname:=word}
```

上面语法的含义是，如果变量`varname`存在且不为空，则返回它的值，否则将它设为`word`，并且返回`word`。它的目的是设置变量的默认值，比如`${count:=0}`表示变量`count`不存在时返回`0`，且将`count`设为`0`。

```
${varname:+word}
```

上面语法的含义是，如果变量名存在且不为空，则返回`word`，否则返回空值。它的目的是测试变量是否存在，比如`${count:+1}`表示变量`count`存在时返回`1`（表示`true`），否则返回空值。

```
${varname:?message}
```

上面语法的含义是，如果变量`varname`存在且不为空，则返回它的值，否则打印出`varname: message`，并中断脚本的执行。如果省略了`message`，则输出默认的信息“parameter null or not set.”。它的目的是防止变量未定义，比如`${count:?"undefined!"}`表示变量`count`未定义时就中断执行，抛出错误，返回给定的报错信息`undefined!`。

上面四种语法如果用在脚本中，变量名的部分可以用到数字`1`到`9`，表示脚本的参数。

```
filename=${1:?"filename missing."}
```

上面代码出现在脚本中，`1`表示脚本的第一个参数。如果该参数不存在，就退出脚本并报错。

## declare 命令

`declare`命令可以声明一些特殊类型的变量，为变量设置一些限制，比如声明只读类型的变量和整数类型的变量。

`declare`命令的主要参数（OPTION）如下。

*   `-a`：声明数组变量。
*   `-f`：输出所有函数定义。
*   `-F`：输出所有函数名。
*   `-i`：声明整数变量。
*   `-l`：声明变量为小写字母。
*   `-p`：查看变量信息。
*   `-r`：声明只读变量。
*   `-u`：声明变量为大写字母。
*   `-x`：该变量输出为环境变量。

`declare`命令如果用在函数中，声明的变量只在函数内部有效，等同于`local`命令。

**`-x`参数**

`-x`参数等同于`export`命令，可以输出一个变量为子 Shell 的环境变量。

```
$ declare -x foo
# 等同于
$ export foo
```

## Reference

- [Bash 变量 - Bash 脚本教程](https://wangdoc.com/bash/variable.html)
