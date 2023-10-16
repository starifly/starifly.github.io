---
title: 阿里云数据源对接grafana
author: starifly
date: 2023-01-26T17:27:06+08:00
lastmod: 2023-01-26T17:27:06+08:00
categories: [grafana]
tags: [grafana,阿里云]
draft: true
slug: alicloud-data-source-docking-with-grafana
---

通过Grafana aliyun_cms_grafana_datasource插件对接查看polardb监控数据

## 安装云监控数据源服务插件

1. 执行以下命令，下载插件到目录/var/lib/grafana/plugins/。

```
cd /var/lib/grafana/plugins/
wget https://github.com/aliyun/aliyun-cms-grafana/releases/download/v2.0/aliyun_cms_grafana_datasource_v2.0.1.tar.gz
```

2. 执行以下命令，将插件解压到目录aliyun_cms_grafana_datasource。

```
tar -xzf aliyun_cms_grafana_datasource_v2.0.tar.gz
```

3. 配置插件

```
#执行以下命令，打开目录/usr/share/grafana/conf中的配置文件defaults.ini。
cd /usr/share/grafana/conf
vi defaults.ini
#配置`allow_loading_unsigned_plugins`为插件的解压目录aliyun_cms_grafana_datasource，允许未签名的云监控数据源服务插件运行。
allow_loading_unsigned_plugins = aliyun_cms_grafana_datasource
```

4. 执行以下命令，重启Grafana服务。

## 创建云监控数据源服务

在Add data source页面，单击最下方的CMS Grafana Service，填写云监控数据源的名称和账号信息，`Aliyun UserId`为阿里云账号ID，`AccessKeyId`为阿里云账号或RAM子用户的AccessKey ID，`AccessKey`为阿里云账号或RAM子用户的AccessKey Secret。推荐使用RAM子用户，并授予如下图所示的权限：

![](/images/aliyun-monitor-01.png)

## 导入dashboard

grafana导入Alibaba Cloud Monitor For Polardb Dashboard，ID为13614。也可以手动导入如下json：

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
  "description": "Alibaba Cloud Monitor For PolarDB Dashboard from CloudMonitor using aliyun-cms-grafana",
  "editable": true,
  "fiscalYearStartMonth": 0,
  "gnetId": 13614,
  "graphTooltip": 0,
  "id": 16,
  "iteration": 1646449081418,
  "links": [],
  "liveNow": false,
  "panels": [
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
      "description": "",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 0,
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_cpu_utilization",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1366",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1367",
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
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 8,
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_active_sessions",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1207",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1208",
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
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 16,
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_connection_utilization",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1289",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1290",
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
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_data_io",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1443",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1444",
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
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_data_iops",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1520",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1521",
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
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_iops",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1597",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1598",
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
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 0,
        "y": 14
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_tps",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1982",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1983",
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
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 8,
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_mem_hit_ratio",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1674",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1675",
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
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 7,
        "w": 8,
        "x": 16,
        "y": 14
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_slow_queries_ps",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1905",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1906",
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
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 8,
        "x": 0,
        "y": 21
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_memory_utilization",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1751",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1752",
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
      "datasource": {
        "type": "aliyun_cms_grafana_datasource",
        "uid": "YQ-nY2Lnz"
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 8,
        "x": 8,
        "y": 21
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
          "describe": "",
          "dimensions": [
            "$dimensions"
          ],
          "group": 62683123,
          "metric": "cluster_qps",
          "period": "300",
          "project": "acs_polardb",
          "refId": "A",
          "target": [
            "Average"
          ],
          "type": "timeserie",
          "xcol": "timestamp",
          "ycol": [
            "Average"
          ]
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
          "$$hashKey": "object:1828",
          "format": "short",
          "logBase": 1,
          "show": true
        },
        {
          "$$hashKey": "object:1829",
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
  "tags": [],
  "templating": {
    "list": [
      {
        "current": {
          "selected": false,
          "text": "All",
          "value": "$__all"
        },
        "datasource": {
          "type": "aliyun_cms_grafana_datasource",
          "uid": "YQ-nY2Lnz"
        },
        "definition": "dimension(acs_polardb, cluster_cpu_utilization,null,null)",
        "hide": 0,
        "includeAll": true,
        "multi": true,
        "name": "dimensions",
        "options": [],
        "query": "dimension(acs_polardb, cluster_cpu_utilization,null,null)",
        "refresh": 1,
        "regex": "",
        "skipUrlSync": false,
        "sort": 0,
        "tagValuesQuery": "",
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      }
    ]
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "5s",
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ]
  },
  "timezone": "",
  "title": "Alibaba Cloud Monitor For Polardb Dashboard",
  "uid": "npqSBWJGz",
  "version": 1,
  "weekStart": ""
}
```

## Reference

- [通过Grafana插件查看监控数据](https://www.alibabacloud.com/help/zh/doc-detail/109434.htm)
- [云数据库PolarDB MySQL（新版）](https://help.aliyun.com/document_detail/165836.htm?spm=a2c4g.11186623.0.0.37361c27v7XCrq#concept-2497787)
- [https://github.com/aliyun/aliyun-cms-grafana](https://github.com/aliyun/aliyun-cms-grafana)
