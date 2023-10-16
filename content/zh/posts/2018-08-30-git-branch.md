---
title: Git分支管理
author: starifly
date: 2018-08-30T21:48:45+08:00
lastmod: 2018-09-02
categories: [git]
tags: [git]
slug: git-branch
---

git的分支管理十分强大，本文主要涉及git中分支的查看和最基本的分支管理操作，主要包括分支的创建、切换、合并、衍合以及分支的推送和拉取等。同时还介绍了如何删除本地分支以及远程分支。

阅读前面的博文有助于理解本文：

1. 《[Git基本操作](https://starifly.github.io/post/git-base/)》
2. 《[Git的撤销更改](https://starifly.github.io/post/git-reset/)》

## 查看分支

```shell
//查看当前仓库的分支，分支前的*表示当前所处的分支
$ git branch

//查看远程仓库的分支
$ git branch -r

//查看所有分支
$ git branch -a

//查看所有分支和该分支上最后的一次提交
$ git branch -v

//查看已经合入当前分支的所有分支
$ git branch --merged

//查看未被合入分支
$ git branch --no-merged

//查看当前分支对应的远程分支（追踪关系）
$ git branch -vv

//可视化目前的分支状态
$ git log --oneline --graph --all
```

## 分支创建和切换

我们可以使用`git branch <分支名>`命令来创建一个分支，然后使用`git checkout <分支名>`来切换分支。

如果觉得麻烦，我们可以使用一个简便的命令创建并切换到该分支上：`git checkout -b <分支名>`。

>也可以使用`git checkout -`命令来切换回上一个分支

## 分支合并

假设当前仓库存在两个分支，我们先切换到master分支，然后通过`git merge --no-ff <分支名>`命令将分支合并到master分支上。

其中`--no-ff`表示强行关闭fast-forward方式，fast-forward方式表示当条件允许时，git直接把HEAD指针指向合并分支的头，完成合并，这种方式合并速度快，但是在整个过程中没有创建commit，所以如果当我们删除掉这个分支时就再也找不回来了，因此在这里我们将之关闭。

如果`git merge`合并分支时出现冲突，先重新编辑冲突文件，编辑完成之后，再执行`git add`和`git commit`即可。

## 分支衍合

衍合（rebase）又称变基，其实也是分支合并的一种方式，既然也是合并，那它跟上文讲的合并（merge）有什么区别呢？

假如我们现在有master和experiment两条分支：

![](https://git-scm.com/book/en/v2/images/basic-rebase-1.png)

现在想把experiment分支合并到master分支上来，最容易的方法是 `merge` 命令。它会把两个分支的最新快照（C3 和 C4）以及二者最近的共同祖先（C2）进行三方比较合并，合并的结果是生成一个新的快照（并提交）。

![](https://git-scm.com/book/en/v2/images/basic-rebase-2.png)

其实，还有一种方法：你可以提取在 （C4）中引入的补丁和修改，然后在（C3）的基础上应用一次。 在git中，这种操作就叫做变基。 你可以使用 `rebase` 命令将提交到某一分支上的所有修改都移至另一分支上，就好像“重新播放”一样。

具体做法：

1.检出experiment分支，对master进行衍合

```shell
$ git checkout experiment
$ git rebase master
```

它的原理是首先找到这两个分支（即当前分支 experiment、变基操作的目标基底分支 master）的最近共同祖先（C2），然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件，然后将当前分支指向目标基底（C3）, 最后以此将之前另存为临时文件的修改依序应用。

![](https://git-scm.com/book/en/v2/images/basic-rebase-3.png)

可以看出，在衍合得过程中，experiment分支会丢弃现有的提交（C4），然后相应地新建一些内容一样但实际上不同的提交（C4’）。

2.再回到master分支，进行一次快进合并

```shell
$ git checkout master
$ git merge experiment
```

![](https://git-scm.com/book/en/v2/images/basic-rebase-4.png)

在衍合过程中有可能会发生冲突，方案有两种：一种是通过`git rebase --abort`命令直接退回到之前的状态，另一种就是解决冲突继续合并。

解决冲突后，要`git add`冲突的文件，标识冲突解决，然后再执行`git rebase --continue`，注意`git add`后不要执行`git commit`。

## 分支的推送和拉取

git push与git pull是一对推送/拉取分支的git命令。  
**git push 使用本地的对应分支来更新对应的远程分支。**

```shell
$ git push <远程主机名> <本地分支名>:<远程分支名>
```
如果省略远程分支名，则表示将本地分支推送与之存在”追踪关系”的远程分支(通常两者同名)，如果该远程分支不存在，则会被新建。

```shell
$ git push origin master
```

上面命令表示，将本地的master分支推送到origin主机的master分支。如果后者不存在，则会被新建。

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支，这条命令是删除远程master分支。

```shell
$ git push origin :<分支名>
//等同于
git push origin --delete <分支名>
```

如果当前分支与远程分支之间存在追踪关系（即分支名相同），则本地分支和远程分支都可以省略。

```shell
$ git push origin
```

上面命令表示，将当前分支推送到origin主机的对应分支。

如果当前分支只有一个追踪分支，那么主机名都可以省略。

```shell
$ git push
```

如果当前分支与多个主机存在追踪关系，则可以使用-u选项指定一个默认主机，这样后面就可以不加任何参数使用`git push`。

```shell
$ git push -u origin master
```

上面命令将本地的master分支推送到origin主机，同时指定origin为默认主机，后面就可以不加任何参数使用`git push`了。

不带任何参数的`git push`，默认只推送当前分支，这叫做simple方式。此外，还有一种matching方式，会推送所有有对应的远程分支的本地分支。Git 2.0版本之前，默认采用matching方法，现在改为默认采用simple方式。如果要修改这个设置，可以采用`git config`命令。

```shell
$ git config --global push.default matching
//或者
$ git config --global push.default simple
```

更详细的push.default属性，请参考《[Git push与pull的默认行为](https://segmentfault.com/a/1190000002783245)》

还有一种情况，就是不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机，这时需要使用–all选项。

```shell
$ git push --all origin
```

上面命令表示，将所有本地分支都推送到origin主机。

**git pull 取回远程主机某个分支的更新，再与本地的指定分支合并。**

`git pull`与`git push`操作的目的相同，但是操作的目标相反。命令格式如下：

```shell
git pull <远程主机名> <远程分支名>:<本地分支名>
```

例如：

```shell
git pull origin master:my-test
```

上面的命令是将origin厂库的master分支拉取并合并到本地的my-test分支上。

如果省略本地分支，则将自动合并到当前所在分支上。如下：

```shell
git pull origin master
```

在某些场合，git会自动在本地分支与远程分支之间，建立一种追踪关系(tracking)。比如，在`git clone`的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的master分支自动”追踪”origin/master分支。

以下也能自动建立追踪关系：  
1. 检出时：`git checkout -b master origin/master`  
2. 推送时：`git push -u origin <远程分支名>`或`git push --set-upstream origin <远程分支名>`

> 注意，上述使用的前提： 
>
- 检出的本地分支必须和远程分支同名才能自动建立追踪关系  
- 推送时，如果远程没有同名关联分支，则会推送失败。

git也允许手动建立追踪关系。

```shell
$ git branch --set-upstream master origin/next
```

上面命令指定master分支追踪origin/next分支。

如果当前分支与远程分支存在追踪关系，`git pull`就可以省略远程分支名。

```shell
$ git pull origin
```

上面命令表示，本地的当前分支自动与对应的origin主机”追踪分支”(remote-tracking branch)进行合并。

如果当前分支只有一个追踪分支，连远程主机名都可以省略。

```shell
$ git pull
```

上面命令表示，当前分支自动与唯一一个追踪分支进行合并。

如果合并需要采用rebase模式，可以使用`--rebase`选项。

```shell
$ git pull --rebase <远程主机名> <远程分支名>:<本地分支名>
```

## 分支的删除

有时候当我们完成功能等分支的开发时，我们会把它合并到上一层分支，此时我们不再需要这个功能分支了，我们可以通过`git branch -d <分支名>`来删除它。

实际上，对分支的删除只是删除的指向该commit号的指针，并不会删除其相关的提交号, 在日志中仍然可以找到之前的commit记录，也仍然可以在该commit上创建新的分支。

如果你还想删除远程分支，要使用`git push origin --delete <分支名>`

## Reference

- <https://segmentfault.com/a/1190000011927868#articleHeader5>
- <https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA>
- <https://www.yiibai.com/git/git_push.html#article-start>
- <https://www.yiibai.com/git/git_pull.html>
