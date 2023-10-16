---
title: 基于jenkins gitlab harbor的k8s CICD
author: starifly
date: 2023-01-19T11:18:20+08:00
lastmod: 2023-01-19T11:18:20+08:00
categories: [k8s,jenkins]
tags: [k8s,jenkins,cicd,gitlab,harbor]
draft: true
slug: jenkins-gitlab-harbor-k8s-cicd
---

本文讲解基于Jenkins gitlab harbor 的k8s CICD流水线建设，其中软件的安装这里省略，java示例代码在本人github上。下面描述的是一些重要的过程和步骤。

## Jenkins需要安装的插件
- Git：gitlab支持
- Pipeline：流水线支持
- Git Parameter：Git参数化构建
- Config File Provider：存储配置文件
- Extended Choice Parameter：扩展选择框参数，支持多选
- Maven Integration：支持maven打包

## 参数化构建

![](/images/jenkins-k8s-01.jpg)

![](/images/jenkins-k8s-02.jpg)

## 流水线

选择流水线脚本或基于scm的流水线脚本，这里选择流水线脚本，脚本如下：

```
pipeline {
	agent any
	
	environment {
		server_name = "java-deployment"
		container_name = "java"
		//image_name = "10.0.4.15:8888/java/repository:${server_name}_${BUILD_NUMBER}"
		image_name = "10.0.4.15:5000/jar/project:${server_name}_${BUILD_NUMBER}"
	}
	
	parameters {
		gitParameter branch: '', branchFilter: '.*', defaultValue: 'origin/develop',
		description: '选择需要的版本', name: 'release', quickFilterEnabled: true,
		selectedValue: 'NONE', sortMode: 'DESCENDING', tagFilter: '*', type: 'PT_REVISION'
		
		choice choices: ['deploy', 'rollback'], description: '''deploy -- 部署
		rollback -- 回滚''', name: 'status'
	}
	
	stages {
		
		stage('拉取代码') {
			steps {
			    // 根据片段生成器checkout生成，这里的release要跟上面参数处关联
				checkout([$class: 'GitSCM', branches: [[name: "${params.release}"]], extensions: [], 
				userRemoteConfigs: [[credentialsId: 'gitlab', url: 'http://10.0.4.15:1480/root/java-project.git']]])
			}
		}
		
		stage('打包编译') {
		    
		    when {
				environment name: 'status', value: 'deploy'
			}
			
			steps {
				sh """
					mvn clean install
				"""
			}
		}
		
		stage('构建上传镜像') {
		    
		    when {
				environment name: 'status', value: 'deploy'
			}
			
			steps {
				sh """
					docker build -t "${image_name}" .
					docker push ${image_name} && docker rmi ${image_name}
				"""
			}
		}
		
		stage('发布到K8S') {
		    
		    when {
				environment name: 'status', value: 'deploy'
			}
			
			steps {
				sh """
					kubectl set image deployment/"${server_name}" "${container_name}"="${image_name}" -n dev
				"""
			}
		}
		
		stage('回滚') {
		    
		    when {
				environment name: 'status', value: 'rollback'
			}
			
			steps {
				sh """
					kubectl rollout undo deployment/${server_name} -n dev
				"""
			}
		}
		
	}
}
```

## 构建测试

![](/images/jenkins-k8s-03.jpg)

如图，先选择需要的版本，再选择部署或回滚，最后点击开始构建即可。

## Reference

- [Jenkins基于阿里云K8s集群（ACK）实现CI/CD](https://www.bilibili.com/video/BV17Z4y127C8)
- [Jenkins Pipeline + Git + Harbor + Kubernetes 部署 Springboot 项目 ](https://www.cnblogs.com/keithtt/p/13149895.html)
