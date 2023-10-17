---
title: 使用grafana+prometheus监控ceph集群并实现钉钉报警
author: starifly
date: 2021-10-06T19:20:04+08:00
lastmod: 2021-10-06T19:20:04+08:00
categories: [ceph]
tags: [ceph,grafana,prometheus,dingding]
draft: false
slug: grafana-prometheus-monitor-ceph-cluster
---

在Ceph Luminous之前的版本，可以使用第三方的Prometheus exporter ceph_exporter。 Ceph Luminous 12.2.1后的mgr中自带了Prometheus插件，内置了 Prometheus ceph exporter，可以使用Ceph mgr内置的exporter作为Prometheus的target。

本系统组件（都是部署在 docker 容器中）如下：

- ceph-nautilus（14.2.20）版本
- grafana的v8.1.3 (a61f38238c)版本
- prometheus的v2.29.2版本
- prometheus-webhook-dingtalk的v0.3.0版本
- alertmanager的0.23.0版本

**【启动 prometheus】**

```bash
ceph mgr module enable prometheus
docker run -d --name=prometheus --net=host --restart=always -v /etc/localtime:/etc/localtime:ro  prom/prometheus:v2.29.2
# 进入 Prometheus 容器中，vi /etc/prometheus/prometheus.yml
  - job_name: "ceph"
    static_configs:
      - targets: ["localhost:9283"]
```

**【启动 grafana】**

`docker run -d --name=grafana --net=host --restart=always -v /etc/localtime:/etc/localtime:ro  grafana/grafana`

添加prometheus数据源，然后再添加dashboard（这里添加id为9550的dashboard）

**【启动钉钉插件】**

先要获取钉钉的 webhook，具体方法可以百度。

`docker run -d --name=dingtalk --net=host --restart=always timonwong/prometheus-webhook-dingtalk:v0.3.0 --ding.profile="<web-hook-name>=<dingtalk-webhook>"`

这里解释一下两个变量：
- web-hook-name ：prometheus-webhook-dingtalk 支持多个钉钉 webhook，不同 webhook 就是靠名字对应到 URL 来做映射的。要支持多个钉钉 webhook，可以用多个 --ding.profile 参数的方式支持，例如：sudo docker run -d --restart always -p 8060:8060 timonwong/prometheus-webhook-dingtalk:v0.3.0 --ding.profile="webhook1=https://oapi.dingtalk.com/robot/send?access_token=token1" --ding.profile="webhook2=https://oapi.dingtalk.com/robot/send?access_token=token2"。而名字和 URL 的对应规则如下，ding.profile="webhook1=......"，对应的 API URL 为：http://localhost:8060/dingtalk/webhook1/send
- dingtalk-webhook：这个就是之前获取的钉钉 webhook。

**【启动 alertmanager】**

`docker run -d --name=alertmanager --net=host --restart=always prom/alertmanager`

添加 webhook 告警，进入容器中，vi /etc/alertmanager/alertmanager.yml

```yml
global:
  resolve_timeout: 5m
 
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'

receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://192.168.5.205:8060/dingtalk/webhook1/send'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

重启容器， `docker restart alertmanager`

配置Prometheus，添加Alertmanager端点，进入 Prometheus 容器中，vi /etc/prometheus/prometheus.yml
```yml
alerting:
  alertmanagers:
  - static_configs:
    - targets: ["192.168.1.10:9093"]
```

添加告警规则文件
```yml
$ cat /etc/prometheus/prometheus.yml
......
rule_files:
    - /etc/prometheus/rules/ceph.yml
```
接下来轮到刚刚提到的告警规则文件了，下面这两个例子，第一个定义了osd状态为down的时候，发出告警消息，第二个定义在 ceph 存储空间使用率大于 80% 的情况下，发出告警消息。
```yml
$ cat /etc/prometheus/rules/ceph.yml
......
groups:
- name: ceph-rule
  rules:
  - alert: Ceph OSD Down
    expr: ceph_osd_up == 0
    for: 2m
    labels:
      product: Ceph测试集群
    annotations:
      Warn: "{{$labels.instance}}: OSD挂掉了"
      Description: "{{$labels.instance}}:{{ $labels.ceph_daemon }}当前状态为{{ $value }}"

  - alert: 集群空间使用率
    expr: ceph_cluster_total_used_bytes / ceph_cluster_total_bytes * 100 > 80
    for: 2m
    labels:
      product: Ceph测试集群
    annotations:
      Warn: "{{$labels.instance}}:集群空间不足"
      Description: "{{$labels.instance}}:当前空间使用率为{{ $value }}"
```

重启容器， `docker restart prometheus`

- [使用 Prometheus 监控 Ceph](https://www.jianshu.com/p/f0fae97d9349)
- [使用prometheus+Grafana监控ceph 集群](https://blog.51cto.com/u_7603402/2446747)
- [https://docs.ceph.com/en/nautilus/mgr/prometheus/#mgr-prometheus](https://docs.ceph.com/en/nautilus/mgr/prometheus/#mgr-prometheus)
- [https://docs.ceph.com/en/nautilus/mgr/dashboard/#enabling-the-embedding-of-grafana-dashboards](https://docs.ceph.com/en/nautilus/mgr/dashboard/#enabling-the-embedding-of-grafana-dashboards)
