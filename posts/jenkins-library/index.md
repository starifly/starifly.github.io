# Jenkins共享库笔记


使用共享库：@Library('pipeline-library-demo@dev')_，其中pipeline-library-demo是Jenkins中配置的共享库名称，dev是共享库的分支名

<!--more-->

1. src下的文件建议以tools.groovy小写方式命名，方法名以PrintMes这种方式命名：

```groovy
package org.devops

def PrintMes(value,color){
	colors = ['red'   : "\033[40;31m >>>>>>>>>>>${value}<<<<<<<<<<< \033[0m",
		  'blue'  : "\033[47;34m ${value} \033[0m",
		  'green' : "[1;32m>>>>>>>>>>${value}>>>>>>>>>>[m",
		  'green1' : "\033[40;32m >>>>>>>>>>>${value}<<<<<<<<<<< \033[0m" ]
	ansiColor('xterm') {
		println(colors[color])
	}
}
```

pipeline调用：在外部定义def tools = new org.devops.tools()，再在流水线中使用tools.PrintMes("获取代码",'green')

2. vars下定义的方法的是全局，也就是在pipeline或者src下都可以直接调用。在pipeline中可以直接通过文件名调用如sayHi 'var'，但是如果一个文件存在多个方法推荐sayHi.call('阳明')这种方式调用以区分不同的方法

vars下的文件命名以驼峰命名，如sayHi.groovy，且方法名也以驼峰方式命名：

```groovy
#!/usr/bin/env groovy

def call(String name='QIKQIAK') {
  echo "Hello, ${name}."
}
```

## Reference

Jenkins - 共享库（Shared Libraries） - https://www.cnblogs.com/anliven/p/13693871.html
Jenkins 共享库使用示例 - https://mp.weixin.qq.com/s/n5pmHYRoAYVbpUs6VadQ_w
实践: 使用Jenkins共享库对流水线进行扩展 - https://mp.weixin.qq.com/s/x3YVXo4geTgEZE-Gga6lkQ
使用Jenkins扩展共享库进行钉钉消息推送 - https://mp.weixin.qq.com/s/svLUitWqAhhHAuYjZgjqmA


---

> 作者: [starifly](https://github.com/starifly)  
> URL: https://starifly.github.io/posts/jenkins-library/  

