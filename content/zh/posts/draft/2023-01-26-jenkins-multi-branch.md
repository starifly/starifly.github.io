---
title: jenkins多分支
author: starifly
date: 2023-01-26T18:48:31+08:00
lastmod: 2023-01-26T18:48:31+08:00
categories: [jenkins]
tags: [jenkins]
draft: true
slug: jenkins-multi-branch
---

对于有多分支的情况，比如master分支对应生产环境，develop分支对应测试环境，此时在jenkins上有两种不同的方式建item，一种是为每一条分支建立一条流水线或自由风格的item，此时视图可以以不同的环境分类，这样每一个环境的构建和发布能够独立开来互不影响；第二种是建立一条多分支流水线item，它会自动去查找代码分支下的Jenkinsfile文件，文件内的内容可以相同，但要根据分支名做好相应的区别处理，此时如果要做webhook，有Generic Webhook Trigger或Multibranch Scan Webhook Trigger两种插件供选择，但是都有弊端，当有事件发生的时候所有分支都会被构建，只能在配置分支源那里过滤掉不需要自动构建的分支，另外如果要手动发布，可以在Jenkinsfile里面以添加如下所示的参数方式发布：

```
    parameters {
        // 提供要部署的服务器选项
        choice(
            description: '你须要选择哪一个环境进行部署 ?',
            name: 'env',
            choices: ['测试环境', '线上环境']
        )
        // 提供构建的模块选项
        choice(
            description: '你须要选择哪一个模块进行构建 ?',
            name: 'moduleName',
            choices: ['kuaimen-contract', 'kuaimen-core', 'kuaimen-eureka-server', 'kuaimen-manage', 'kuaimen-member', 'kuaimen-order', 'kuaimen-shop', 'tiemuzhen-manage']
        )
        booleanParam(name: 'isAll', defaultValue: false, description: '是否须要全量（包含clean && build）
')
        string(name: 'update', defaultValue: '', description: '本次更新内容?')
    }
```

总的来说，在Jenkins上，针对多分支项目不管建立单独的item或者多分支流水线都有优缺点，所以需要根据实际情况灵活选择。
