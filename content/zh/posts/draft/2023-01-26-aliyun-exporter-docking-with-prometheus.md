---
title: aliyun-exporter对接prometheus
author: starifly
date: 2023-01-26T17:38:47+08:00
lastmod: 2023-01-26T17:38:47+08:00
categories: [prometheus]
tags: [prometheus,阿里云]
draft: true
slug: aliyun-exporter-docking-with-prometheus
---

## 安装exporter

需要Python 3.7以上版本

```
pip3 install aliyun-exporter-czb
```

## 使用

首先需要在配置文件中写明阿里云的 Access Key 以及需要拉取的云监控指标，例子如下：

```
credential:
  access_key_id: <YOUR_ACCESS_KEY_ID>
  access_key_secret: <YOUR_ACCESS_KEY_SECRET>
  region_id: <REGION_ID>

metrics:
  acs_polardb:
  - name: cluster_cpu_utilization
    period: 300
  - name: cluster_active_sessions
    period: 300
  - name: cluster_connection_utilization
    period: 300
  - name: cluster_data_io
    period: 300
  - name: cluster_data_iops
    period: 300
  - name: cluster_iops
    period: 300
  - name: cluster_tps
    period: 300
  - name: cluster_mem_hit_ratio
    period: 300
  - name: cluster_slow_queries_ps
    period: 300
  - name: cluster_memory_utilization
    period: 300
  - name: cluster_qps
    period: 300
#metrics:
#  acs_mongodb:
#  - name: CPUUtilization
#    period: 300

info_metrics:
  - polardb
```

阿里云账号需要授予相应权限，比如要获取polardb数据要开通如图所示的权限

![](/images/aliyun-prometheus-01.png)

如何是多个regionid想在一个metrics里呈现,可以配置下面的选项

```
do_info_region:
  - "cn-zhangjiakou"
  - "cn-beijing"
```

启动 Exporter

```
aliyun-exporter -p 9525 -c aliyun-exporter.yml
```

访问 localhost:9525/metrics 查看指标抓取是否成功

prometheus增加配置：

```
  - job_name: 'aliyun-exporter'
    static_configs:
      - targets:
        - 192.168.1.100:9525
```

## grafana dashboard

导入polardb配置：

```
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "target": {
          "limit": 100,
          "matchAny": false,
          "tags": [],
          "type": "dashboard"
        },
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "fiscalYearStartMonth": 0,
  "graphTooltip": 0,
  "id": 18,
  "links": [],
  "liveNow": false,
  "panels": [
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 0,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 2,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_cpu_utilization[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "CPU使用率(%)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:55",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:56",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 8,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 4,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_active_sessions[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "活跃连接数(个)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:377",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:378",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 16,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 6,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_connection_utilization[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "连接数使用率(%)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:774",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:775",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 0,
        "y": 7
      },
      "hiddenSeries": false,
      "id": 8,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_data_io[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "每秒存储引擎IO吞吐量(KB)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:1179",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1180",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 8,
        "y": 7
      },
      "hiddenSeries": false,
      "id": 10,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_data_iops[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "每秒存储引擎IO次数(次/秒)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:3921",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:3922",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 16,
        "y": 7
      },
      "hiddenSeries": false,
      "id": 12,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_iops[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "每秒IO次数(次/秒)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:4006",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:4007",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 0,
        "y": 14
      },
      "hiddenSeries": false,
      "id": 14,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_tps[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "每秒事务数(个/秒)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:1754",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1755",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 8,
        "y": 14
      },
      "hiddenSeries": false,
      "id": 16,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_mem_hit_ratio[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "内存命中率(%)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:2080",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:2081",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 16,
        "y": 14
      },
      "hiddenSeries": false,
      "id": 18,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_slow_queries_ps[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "每秒慢查询数量(个/秒)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:2481",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:2482",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 8,
        "x": 0,
        "y": 21
      },
      "hiddenSeries": false,
      "id": 20,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_memory_utilization[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "内存使用率(%)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:2961",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:2962",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 8,
        "x": 8,
        "y": 21
      },
      "hiddenSeries": false,
      "id": 22,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "8.3.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "w8OggWYnz"
          },
          "exemplar": true,
          "expr": "avg_over_time(aliyun_acs_polardb_cluster_qps[5m])",
          "interval": "",
          "legendFormat": "{{clusterId}}_{{nodeId}}.average",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeRegions": [],
      "title": "每秒查询数量(个)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "mode": "time",
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:3362",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:3363",
          "format": "short",
          "logBase": 1,
          "show": true
        }
      ],
      "yaxis": {
        "align": false
      }
    }
  ],
  "schemaVersion": 34,
  "style": "dark",
  "tags": [
    "polardb",
    "database"
  ],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "",
  "title": "Polardb dashboard",
  "uid": "npqSBWJGz",
  "version": 3,
  "weekStart": ""
}
```

- [https://github.com/hanyunfeng6163/aliyun-exporter](https://github.com/hanyunfeng6163/aliyun-exporter)
