---
title: k8s之ConfigMap详细理解及使用
author: starifly
date: 2023-01-28T15:43:49+08:00
lastmod: 2023-01-28T15:43:49+08:00
categories: [k8s]
tags: [k8s,configmap]
draft: true
slug: k8s-configmap
---

## ConfigMap介绍

ConfigMap是一种API对象，用来将非加密数据保存到键值对中。可以用作环境变量、命令行参数或者存储卷中的配置文件。

ConfigMap可以将环境变量配置信息和容器镜像解耦，便于应用配置的修改。如果需要存储加密信息时可以使用Secret对象。

## ConfigMap创建

### 通过命令行创建configmap

可以使用 kubectl create configmap 从文件、目录或者 key-value 字符串创建等创建 ConfigMap

1. 通过文件创建configmap

```
[root@node1 ~]# cat test.txt 
key1=hello
key2=world

[root@node1 ~]# kubectl create configmap my-config --from-env-file=test.txt
configmap/my-config created

[root@node1 ~]# kubectl describe configmap my-config
Name:         my-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
key1:
----
hello
key2:
----
world

BinaryData
====

Events:  <none>
```

看到该configmap中有两个键值对，key1:hello 和 key2:world

2. 通过文件夹创建configmap

```
[root@node1 ~]# mkdir config
[root@node1 ~]# echo hello > config/test1
[root@node1 ~]# echo world > config/test2
```

根据文件夹创建configmap

```
[root@node1 ~]# kubectl create configmap dir-config --from-file=config/
configmap/dir-config created

[root@node1 ~]# kubectl describe configmap dir-config
Name:         dir-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
test1:
----
hello

test2:
----
world


BinaryData
====

Events:  <none>
```

看到该configmap资源中有两个键值对，test1:hello和test2:world，key为文件名，value为文件内容

3. 通过键值对创建configmap

```
kubectl create configmap literal-config --from-literal=key1=hello --from-literal=key2=world
kubectl describe configmap literal-config
```

### 通过yaml文件创建

```
#config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
 name: my-config
data:
 key1: hello
 key2: world
```

```
$ kubectl create -f config.yaml
```

## ConfigMap的使用

Pod的使用方式：

- 将ConfigMap中的数据设置为容器的环境变量
- 将ConfigMap中的数据设置为命令行参数
- 使用Volume将ConfigMap作为文件或目录挂载
- 编写代码在 Pod 中运行，使用 Kubernetes API 来读取 ConfigMap

### 配置到容器的环境变量

```
# test-pod-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
 name: test-pod-configmap
spec:
 containers:
  - name: test-busybox
    image: busybox
    imagePullPolicy: IfNotPresent
    args:
    - sleep
    - "86400"
    env:
    - name: KEY1
      valueFrom:
       configMapKeyRef:
        name: my-config
        key: key1
    - name: KEY2
      valueFrom:
       configMapKeyRef:
        name: my-config
        key: key2
```

创建pod

```
$ kubectl create -f test-pod-configmap.yaml
```

### 设置为命令行参数

```
# test-pod-configmap-cmd
apiVersion: v1
kind: Pod
metadata:
 name: test-pod-configmap-cmd
spec:
 containers:
  - name: test-busybox
    image: busybox
    imagePullPolicy: IfNotPresent
    command: [ "/bin/sh","-c","echo $(KEY1) $(KEY2)"]
    env:
    - name: KEY1
      valueFrom:
       configMapKeyRef:
        name: my-config
        key: key1
    - name: KEY2
      valueFrom:
       configMapKeyRef:
        name: my-config
        key: key2
 restartPolicy: Never
```

创建pod，该pod成功启动后会输出环境变量KEY1和KEY2的值

```
$ kubectl create -f test-pod-configmap-cmd.yaml 

#查看pod的日志
$ kubectl logs test-pod-configmap-cmd

#查看环境变量
kubectl exec -it test-pod-configmap -- env | grep KEY
```

### 将configmap挂载到容器中

```
# test-pod-projected-configmap-volume.yaml
apiVersion: v1
kind: Pod
metadata:
 name: test-pod-projected-configmap-volume
spec:
 containers:
 - name: test-pod-busybox
   image: busybox
   imagePullPolicy: IfNotPresent
   args:
   - sleep
   - "86400"
   volumeMounts:
   - name: config-volume
     mountPath: "/projected-volume"
     readOnly: true
 volumes:
 - name: config-volume
   projected:
    sources:
    - configMap:
       name: my-config
```

创建pod

```
$ kubectl create -f test-pod-projected-configmap-volume.yaml
```

进入容器，查看挂在到容器中的文件内容

```
$ kubectl exec -it test-pod-projected-configmap-volume -- /bin/sh
[root@node1 ~]# kubectl exec -it test-pod-projected-configmap-volume -- /bin/sh
/ # cd projected-volume/
/projected-volume # ls
key1  key2
/projected-volume # cat -n key1
     1	hello
/projected-volume # cat -n key2
     1	world
```

> 通过volume挂载和环境变量的区别：  
> 通过Volume挂载到容器内部时，当该configmap的值发生变化时，容器内部具备自动更新的能力，但是通过环境变量设置到容器内部该值不具备自动更新的能力。

## 类文件键

下面的文件configmap-demo.yaml定义了一个ConfigMap，其中包含了两种键值对数据：类属性键和类文件键。

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # 类属性键；每一个键都映射到一个简单的值
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # 类文件键，|表示有多行数据
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true 
```

创建ConfigMap并检查：

```
[root@node1 configmap]# kubectl apply -f configmap-demo.yaml
configmap/game-demo created
[root@node1 configmap]# 
[root@node1 configmap]# kubectl get configmap
NAME               DATA   AGE
dir-config         2      23m
game-demo          4      8s
kube-root-ca.crt   1      47d
my-config          2      19m
```

在Pod中引用ConfigMap，下面的文件configmap-demo-pod.yaml定义了一个Pod，其中spec字段中通过容器环境变量和挂载configMap只读卷两种方式，引用了上面定义的ConfigMap中存储的键值对数据。

```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # 通过容器环境变量来引用ConfigMap中的数据
        - name: PLAYER_INITIAL_LIVES     # 自定义的环境变量名称
          valueFrom:
            configMapKeyRef:
              name: game-demo            # 要获取数据来源的ConfigMap名称
              key: player_initial_lives  # 需要从ConfigMap中取值的键
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"   # ConfigMap中的数据在容器中的挂载路径
        readOnly: true         # 必须为只读卷
  volumes:
    # 通过挂载类型为configMap的只读卷来引用ConfigMap中的数据
    - name: config
      configMap:
        name: game-demo     # 要获取数据来源的ConfigMap名称
        # 来自ConfigMap的一组键，将被创建为文件
        items:
        - key: "game.properties"     # 需要从ConfigMap中取值的键
          path: "game.properties"    # 在volumeMounts.mountPath下挂载的路径名
        - key: "user-interface.properties"
          path: "user-interface.properties"
```

创建上面示例中的Pod，并检查配置数据：


```
[root@node1 configmap]# kubectl apply -f configmap-demo-pod.yaml
pod/configmap-demo-pod created
[root@node1 configmap]# 
[root@node1 configmap]# kubectl exec -it configmap-demo-pod -- sh -c 'echo $PLAYER_INITIAL_LIVES'
3
[root@node1 configmap]# kubectl exec -it configmap-demo-pod -- sh -c 'echo $UI_PROPERTIES_FILE_NAME'
user-interface.properties
[root@node1 configmap]# 
[root@node1 configmap]# kubectl exec -it configmap-demo-pod -- sh -c 'ls /config'
game.properties            user-interface.properties
[root@node1 configmap]# 
[root@node1 configmap]# kubectl exec -it configmap-demo-pod -- sh -c 'cat /config/game.properties'
enemy.types=aliens,monsters
player.maximum-lives=5    
[root@node1 configmap]#
[root@node1 configmap]# kubectl exec -it configmap-demo-pod -- sh -c 'cat /config/user-interface.properties'
color.good=purple
color.bad=yellow
allow.textmode=true
```

上面的例子定义了一个卷并将它作为/config文件夹挂载到demo容器内，然后创建了两个文件/config/game.properties和/config/user-interface.properties，尽管ConfigMap中包含了四个键。这是因为Pod定义中在volumes字段下指定了一个items数组。如果我们不指定items数组，则ConfigMap中的每个键都会变成一个与该键同名的文件，因此会得到四个文件。

## Reference

- [k8s之ConfigMap详细理解及使用](https://blog.csdn.net/skh2015java/article/details/109228836)
- [k8s配置文件数据存储：ConfigMap](https://blog.csdn.net/Sebastien23/article/details/127676298)
