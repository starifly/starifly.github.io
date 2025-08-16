# K8s Cronjob执行时区问题


使用k8s中的cronjob备份数据库发现一个问题，设置一个固定的时间点执行job，但是到了设定的时间后却没有执行。

经过查资料才得知，是k8s中时区的问题，所以需要修改`/etc/kubernetes/manifests/kube-scheduler.yaml`配置文件，增加相应的时区设置：

<!--more-->

```yml
    volumeMounts:
    -  name: localtime
       mountPath: /etc/localtime
       readOnly: true
   volumes:
    - name: localtime
       hostPath:
           path: /etc/localtime
```

再重启k8s，就可以解决

```sh
systemctl restart kubelet
```

## Reference

- [k8s时区问题解决方案]([k8s时区问题解决方案 - 码农教程](http://www.manongjc.com/detail/53-xjvslbtaxzxmpuw.html))
- [kubernetes cronjob 时区问题_k8s cronjob 时区_Magiceses的博客-CSDN博客](https://blog.csdn.net/weixin_43700106/article/details/118217172)


---

> 作者: [starifly](https://github.com/starifly)  
> URL: https://starifly.github.io/posts/k8s-cronjob-time-zone/  

