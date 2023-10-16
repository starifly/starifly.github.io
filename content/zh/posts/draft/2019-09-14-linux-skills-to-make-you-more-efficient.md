---
title: 让你提高效率的linux技巧
author: starifly
date: 2019-09-14T12:42:06+08:00
lastmod: 2020-05-18
categories: [linux]
tags: [linux]
draft: true
slug: linux-skills-to-make-you-more-efficient
---

## 命令编辑及光标移动

- ctrl + u ：删除从开头到光标处的命令文本
- ctrl + K ：删除从光标到结尾处的命令文本
- ctrl + a ：光标移动到命令开头
- ctrl + e ：光标移动到命令结尾
- ctrl + w ：剪切一个词（以空格隔开的字符串）
- alt + f ：光标向前移动一个单词
- alt + b ：光标向后移动一个单词


## 命令行下的复制粘贴

我们知道，在命令行下，复制不能再是 ctrl + c 了，因为它表示终止当前进程，而控制台下的复制粘贴需要使用如下快捷键：

ctrl + insert  
shift + insert


## 屏幕冻结

程序运行时，终端可能输出大量的日志，此时你想查看之前的打印，终端会自动跳到最后，此时可以使用 ctrl + s 键冻结屏幕，使得日志不再继续输出，而如果想要恢复，可使用 ctrl + q 退出冻结。

## 命令的替换

可以使用 ^ 字符实现对上一个命令的文本替换并重新执行命令，例如 ^before^after^ 相当于把上一个命令中的 before 替换为 after 然后重新执行一次。

> $eho helloworld  <==错误的命令
> 
> Command'eho'notfound,did youmean:
> 
> command'echo'from deb coreutils
> 
> command'who'from deb coreutils
> 
> Try:sudo aptinstall<debname\>
> 
> $^e^ec^        <==替换
> 
> echo hello world
> 
> helloworld


## 重复执行历史命令

### ctrl + r

ctrl + r 这个快捷键可以让你搜索之前输入过的所有的命令。假如我现在要重复执行 `uname -a` ，我们可以先按一下 ctrl + r 这个组合键，然后再依次敲入 'u'n'a'，如果之前输入的命令比较少的话，基本只需敲一个 u 或 n 就可以找到你输入的命令了。

```shell
(reverse-i-search)`u': uname -a`
```

找到之后，再敲一下回车，即可重复执行你要输入的命令了。

假如你要对找到的那条命令进行一些小修改，那么只需敲一下左或右的箭头即可。

### ctrl+p 或 ctrl+n 配合 ctrl+o

通过 ctrl+p/n 的组合键找到想要的命令之后，我们可以按 ctrl+o 来执行这条命令。这个组合键与回车不太相同，因为它除了回车之外，还另外跳转到最后一条命令。通过不断地 ctrl+p/n ，然后 ctrl+o ，就可以高效的重复执行你要执行的命令了。

## 获取命令的最后一个参数

!$ 可以获取一条命令的最后一个参数，对于路径很好用。

```shell
$ ls -l /var
$ cd !$  <==会切换到 /var 目录
```

## 软件包管理

升级软件可以使用命令 `rpm -U soft_update.rpm`

## 其它命令

- locate ： 通过名字来查找文件
- 查找包含不规范字符的路径名：查找包含不规范字符的路径名

## Reference

- [5 种方法重复执行历史命令](https://mp.weixin.qq.com/s/2B-8XfCyfrcD2lTKG_0JLA)
- [让你提高效率的 Linux 技巧](https://mp.weixin.qq.com/s/c3l86BvfyR0HOo_c66LtMw)
