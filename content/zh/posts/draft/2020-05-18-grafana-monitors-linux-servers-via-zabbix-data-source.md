---
title: grafana通过zabbix数据源监控linux servers
author: starifly
date: 2020-05-18T21:42:46+08:00
lastmod: 2020-05-18T21:42:46+08:00
categories: [grafana]
tags: [grafana,zabbix,linux]
draft: true
slug: grafana-monitors-linux-servers-via-zabbix-data-source
---

## grafana模板

```
{
  "annotations": {
    "list": [
      {
        "$$hashKey": "object:22",
        "builtIn": 1,
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "description": "Zabbix agent use template Linux servers (Zabbix v4.4.0)",
  "editable": true,
  "gnetId": 11209,
  "graphTooltip": 0,
  "id": 23,
  "iteration": 1588933286689,
  "links": [],
  "panels": [
    {
      "content": "<h1 style=\"text-align: center;\"><img src=\"https://images.esp-online.com/thumbnails/aws_solution_list_thumbnail/2614/9455/9402/ZABBIX_LOGO.jpg\" alt=\"\" width=\"191\" height=\"50\" /></h1>\n<h1 style=\"text-align: center;\"><span style=\"color: #ffffff;\"><strong>Trangnth</strong></span></h1>",
      "datasource": null,
      "gridPos": {
        "h": 4,
        "w": 6,
        "x": 0,
        "y": 0
      },
      "height": "",
      "id": 20,
      "links": [],
      "mode": "html",
      "title": "",
      "transparent": true,
      "type": "text"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": true,
      "colors": [
        "rgba(50, 172, 45, 0.97)",
        "#F2495C",
        "rgba(245, 54, 54, 0.9)"
      ],
      "datasource": "Zabbix",
      "decimals": null,
      "editable": true,
      "error": false,
      "format": "s",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": false,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 6,
        "y": 0
      },
      "height": "",
      "id": 6,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "pluginVersion": "6.1.6",
      "postfix": "",
      "postfixFontSize": "50%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": false
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": "General"
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "System uptime"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "",
      "title": "Server uptime",
      "transparent": true,
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [],
      "valueName": "current"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": true,
      "colors": [
        "#5794F2",
        "#1F60C4",
        "#5794F2"
      ],
      "datasource": "Zabbix",
      "description": "",
      "editable": true,
      "error": false,
      "format": "none",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": false,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 10,
        "y": 0
      },
      "height": "",
      "id": 18,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "postfix": "",
      "postfixFontSize": "80%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": false
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": "$app"
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "Agent ping"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "",
      "title": "Zabbix Agent",
      "transparent": true,
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [
        {
          "op": "=",
          "text": "UP",
          "value": "1"
        },
        {
          "op": "=",
          "text": "DOWN",
          "value": "0"
        }
      ],
      "valueName": "current"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": true,
      "colors": [
        "rgba(50, 172, 45, 0.97)",
        "#FADE2A",
        "rgba(245, 54, 54, 0.9)"
      ],
      "datasource": "Zabbix",
      "description": "",
      "editable": true,
      "error": false,
      "format": "none",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": false,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 13,
        "y": 0
      },
      "height": "",
      "hideTimeOverride": true,
      "id": 22,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "postfix": "",
      "postfixFontSize": "80%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": false
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": ""
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "CPU num"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "",
      "timeFrom": "24h",
      "timeShift": null,
      "title": "Number of CPUs",
      "transparent": true,
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [
        {
          "op": "=",
          "text": "N/A",
          "value": "null"
        }
      ],
      "valueName": "current"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": true,
      "colors": [
        "rgba(50, 172, 45, 0.97)",
        "#FF780A",
        "rgba(245, 54, 54, 0.9)"
      ],
      "datasource": "Zabbix",
      "description": "",
      "editable": true,
      "error": false,
      "format": "bytes",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": false,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 16,
        "y": 0
      },
      "height": "",
      "id": 23,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "pluginVersion": "6.1.6",
      "postfix": "",
      "postfixFontSize": "80%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": false
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": "Memory"
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "Total memory"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "",
      "title": "Memory Total",
      "transparent": true,
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [
        {
          "op": "=",
          "text": "N/A",
          "value": "null"
        }
      ],
      "valueName": "current"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": true,
      "colors": [
        "rgba(50, 172, 45, 0.97)",
        "#37872D",
        "rgba(245, 54, 54, 0.9)"
      ],
      "datasource": "Zabbix",
      "description": "",
      "editable": true,
      "error": false,
      "format": "none",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": false,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 20,
        "y": 0
      },
      "height": "",
      "id": 11,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "postfix": "",
      "postfixFontSize": "80%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": false
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": ""
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "Number of processes"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "",
      "title": "Number of processes",
      "transparent": true,
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [
        {
          "op": "=",
          "text": "N/A",
          "value": "null"
        }
      ],
      "valueName": "current"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": false,
      "colors": [
        "rgba(40, 237, 73, 0.89)",
        "rgba(237, 129, 40, 0.89)",
        "rgba(245, 54, 54, 0.9)"
      ],
      "datasource": "Zabbix",
      "decimals": 0,
      "editable": true,
      "error": false,
      "format": "percent",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": true,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 0,
        "y": 4
      },
      "id": 3,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "postfix": "",
      "postfixFontSize": "50%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": true
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": "Memory"
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "Memory used percent"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "80,90",
      "title": "Memory utilization",
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [
        {
          "op": "=",
          "text": "N/A",
          "value": "null"
        }
      ],
      "valueName": "avg"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": false,
      "colors": [
        "rgba(40, 237, 73, 0.89)",
        "rgba(237, 129, 40, 0.89)",
        "rgba(245, 54, 54, 0.9)"
      ],
      "datasource": "Zabbix",
      "decimals": 2,
      "editable": true,
      "error": false,
      "format": "percent",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": true,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 4,
        "y": 4
      },
      "id": 8,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "postfix": "",
      "postfixFontSize": "50%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": true
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": "CPU"
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "CPU used percent"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": true
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "80,90",
      "timeFrom": null,
      "timeShift": null,
      "title": "CPU utilization",
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [
        {
          "op": "=",
          "text": "N/A",
          "value": "null"
        }
      ],
      "valueName": "current"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": false,
      "colors": [
        "#299c46",
        "rgba(237, 129, 40, 0.89)",
        "#d44a3a"
      ],
      "datasource": "Zabbix",
      "decimals": null,
      "format": "none",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": false,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 8,
        "y": 4
      },
      "id": 27,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "pluginVersion": "6.1.6",
      "postfix": "",
      "postfixFontSize": "50%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": true
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": "CPU"
          },
          "countTriggers": true,
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "Processor load (15 min average per core)"
          },
          "minSeverity": 3,
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "",
      "timeFrom": null,
      "timeShift": null,
      "title": "System load 15 min",
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [
        {
          "op": "=",
          "text": "N/A",
          "value": "null"
        }
      ],
      "valueName": "current"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": false,
      "colors": [
        "#299c46",
        "rgba(237, 129, 40, 0.89)",
        "#d44a3a"
      ],
      "datasource": "Zabbix",
      "format": "percent",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": true,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 12,
        "y": 4
      },
      "id": 13,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "postfix": "",
      "postfixFontSize": "50%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": true
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": "Filesystems"
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "Free disk space on / (percentage)"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "",
      "timeFrom": null,
      "timeShift": null,
      "title": "Space utilization in % for /",
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [
        {
          "op": "=",
          "text": "N/A",
          "value": "null"
        }
      ],
      "valueName": "current"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": false,
      "colors": [
        "#299c46",
        "rgba(237, 129, 40, 0.89)",
        "#d44a3a"
      ],
      "datasource": "Zabbix",
      "format": "none",
      "gauge": {
        "maxValue": 50,
        "minValue": 0,
        "show": true,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 16,
        "y": 4
      },
      "id": 28,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "postfix": "",
      "postfixFontSize": "50%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": false
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": ""
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "Number of logged in users"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "",
      "timeFrom": null,
      "timeShift": null,
      "title": "Number of logged in users",
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [
        {
          "op": "=",
          "text": "N/A",
          "value": "null"
        }
      ],
      "valueName": "current"
    },
    {
      "cacheTimeout": null,
      "colorBackground": false,
      "colorValue": false,
      "colors": [
        "#299c46",
        "rgba(237, 129, 40, 0.89)",
        "#d44a3a"
      ],
      "datasource": "Zabbix",
      "format": "none",
      "gauge": {
        "maxValue": 100,
        "minValue": 0,
        "show": true,
        "thresholdLabels": false,
        "thresholdMarkers": true
      },
      "gridPos": {
        "h": 6,
        "w": 4,
        "x": 20,
        "y": 4
      },
      "id": 29,
      "interval": null,
      "links": [],
      "mappingType": 1,
      "mappingTypes": [
        {
          "name": "value to text",
          "value": 1
        },
        {
          "name": "range to text",
          "value": 2
        }
      ],
      "maxDataPoints": 100,
      "nullPointMode": "connected",
      "nullText": null,
      "postfix": "",
      "postfixFontSize": "50%",
      "prefix": "",
      "prefixFontSize": "50%",
      "rangeMaps": [
        {
          "from": "null",
          "text": "N/A",
          "to": "null"
        }
      ],
      "sparkline": {
        "fillColor": "rgba(31, 118, 189, 0.18)",
        "full": false,
        "lineColor": "rgb(31, 120, 193)",
        "show": false
      },
      "tableColumn": "",
      "targets": [
        {
          "application": {
            "filter": ""
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "Number of running processes"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": "",
      "timeFrom": null,
      "timeShift": null,
      "title": "Number of running processes",
      "type": "singlestat",
      "valueFontSize": "80%",
      "valueMaps": [
        {
          "op": "=",
          "text": "N/A",
          "value": "null"
        }
      ],
      "valueName": "current"
    },
    {
      "aliasColors": {
        "CPU iowait time": "#F2C96D",
        "CPU system time": "#65C5DB",
        "CPU user time": "#E24D42"
      },
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "Zabbix",
      "editable": true,
      "error": false,
      "fill": 1,
      "fillGradient": 0,
      "grid": {},
      "gridPos": {
        "h": 16,
        "w": 12,
        "x": 0,
        "y": 10
      },
      "height": "610",
      "hiddenSeries": false,
      "id": 21,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 2,
      "links": [],
      "nullPointMode": "connected",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 5,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "application": {
            "filter": "CPU"
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "hide": false,
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "/^CPU/"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "CPU usage",
      "tooltip": {
        "msResolution": true,
        "shared": true,
        "sort": 0,
        "value_type": "cumulative"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:358",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:359",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "ackEventColor": "rgba(0, 0, 0, 0)",
      "ageField": true,
      "customLastChangeFormat": false,
      "datasource": null,
      "descriptionAtNewLine": false,
      "descriptionField": true,
      "fontSize": "100%",
      "gridPos": {
        "h": 9,
        "w": 12,
        "x": 12,
        "y": 10
      },
      "highlightBackground": false,
      "highlightNewEvents": false,
      "highlightNewerThan": "1h",
      "hostField": true,
      "hostGroups": false,
      "hostProxy": false,
      "hostTechNameField": false,
      "hostsInMaintenance": true,
      "id": 17,
      "lastChangeFormat": "",
      "layout": "table",
      "limit": 10,
      "links": [],
      "markAckEvents": false,
      "okEventColor": "#0a437c",
      "pageSize": 10,
      "problemTimeline": true,
      "resizedColumns": [],
      "schemaVersion": 7,
      "severityField": false,
      "showEvents": {
        "text": "Problems",
        "value": 1
      },
      "showTags": true,
      "showTriggers": "all triggers",
      "sortTriggersBy": {
        "text": "last change",
        "value": "lastchange"
      },
      "statusField": true,
      "statusIcon": false,
      "targets": [
        {
          "$$hashKey": "object:185",
          "application": {
            "filter": ""
          },
          "datasource": "Zabbix",
          "group": {
            "filter": ""
          },
          "host": {
            "filter": ""
          },
          "proxy": {
            "filter": ""
          },
          "refId": "B",
          "tags": {
            "filter": ""
          },
          "trigger": {
            "filter": ""
          }
        }
      ],
      "title": "Zabbix Server Events in Group Linux servers",
      "triggerSeverity": [
        {
          "$$hashKey": "object:136",
          "color": "#B7DBAB",
          "priority": 0,
          "severity": "Not classified",
          "show": true
        },
        {
          "$$hashKey": "object:137",
          "color": "#82B5D8",
          "priority": 1,
          "severity": "Information",
          "show": true
        },
        {
          "$$hashKey": "object:138",
          "color": "#E5AC0E",
          "priority": 2,
          "severity": "Warning",
          "show": true
        },
        {
          "$$hashKey": "object:139",
          "color": "#C15C17",
          "priority": 3,
          "severity": "Average",
          "show": true
        },
        {
          "$$hashKey": "object:140",
          "color": "#BF1B00",
          "priority": 4,
          "severity": "High",
          "show": true
        },
        {
          "$$hashKey": "object:141",
          "color": "#890F02",
          "priority": 5,
          "severity": "Disaster",
          "show": true
        }
      ],
      "type": "alexanderzobnin-zabbix-triggers-panel"
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "Zabbix",
      "editable": true,
      "error": false,
      "fill": 3,
      "fillGradient": 0,
      "grid": {},
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 12,
        "y": 19
      },
      "hiddenSeries": false,
      "id": 25,
      "legend": {
        "alignAsTable": false,
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "rightSide": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 2,
      "links": [],
      "maxPerRow": 3,
      "nullPointMode": "connected",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 5,
      "points": false,
      "renderer": "flot",
      "repeat": null,
      "repeatDirection": "h",
      "seriesOverrides": [
        {
          "alias": "/Incoming/",
          "transform": "negative-Y"
        }
      ],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "application": {
            "filter": "$app"
          },
          "countTriggers": true,
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "hide": false,
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "$netif"
          },
          "minSeverity": 3,
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Network traffic on $netif",
      "tooltip": {
        "msResolution": false,
        "shared": true,
        "sort": 0,
        "value_type": "cumulative"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "bps",
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {
        "Available memory": "#BADFF4"
      },
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "Zabbix",
      "decimals": 2,
      "editable": true,
      "error": false,
      "fill": 2,
      "fillGradient": 0,
      "grid": {},
      "gridPos": {
        "h": 8,
        "w": 6,
        "x": 0,
        "y": 26
      },
      "hiddenSeries": false,
      "id": 2,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "hideEmpty": false,
        "hideZero": false,
        "max": true,
        "min": true,
        "rightSide": false,
        "show": true,
        "sideWidth": 250,
        "sort": null,
        "sortDesc": null,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 2,
      "links": [],
      "maxDataPoints": "",
      "nullPointMode": "connected",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 1,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [
        {
          "alias": "Available memory",
          "yaxis": 1
        }
      ],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "application": {
            "filter": "Memory"
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "hide": false,
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "Available memory"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Free Memory",
      "tooltip": {
        "msResolution": true,
        "shared": true,
        "sort": 0,
        "value_type": "cumulative"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "bytes",
          "label": "Memória",
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": "",
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {
        "Zabbix queue": "#BF1B00"
      },
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "Zabbix",
      "editable": true,
      "error": false,
      "fill": 1,
      "fillGradient": 0,
      "grid": {},
      "gridPos": {
        "h": 8,
        "w": 6,
        "x": 6,
        "y": 26
      },
      "hiddenSeries": false,
      "id": 9,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "hideEmpty": false,
        "hideZero": false,
        "max": true,
        "min": true,
        "rightSide": false,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 2,
      "links": [],
      "nullPointMode": "connected",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 5,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "application": {
            "filter": "Zabbix server"
          },
          "functions": [
            {
              "$$hashKey": "object:236",
              "def": {
                "category": "Transform",
                "defaultParams": [
                  100
                ],
                "name": "scale",
                "params": [
                  {
                    "name": "factor",
                    "options": [
                      100,
                      0.01,
                      10,
                      -1
                    ],
                    "type": "float"
                  }
                ]
              },
              "params": [
                "100"
              ],
              "text": "scale(100)"
            }
          ],
          "group": {
            "filter": "Zabbix servers"
          },
          "host": {
            "filter": "192.168.5.59_王总测试"
          },
          "item": {
            "filter": "Number of processed values per second"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        },
        {
          "application": {
            "filter": "Zabbix server"
          },
          "functions": [],
          "group": {
            "filter": "Zabbix servers"
          },
          "host": {
            "filter": "SVMGBZABPRD"
          },
          "item": {
            "filter": "Zabbix queue"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "B",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Zabbix Server Performance",
      "tooltip": {
        "msResolution": true,
        "shared": true,
        "sort": 0,
        "value_type": "cumulative"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "Zabbix",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 6,
        "x": 12,
        "y": 26
      },
      "hiddenSeries": false,
      "id": 14,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 2,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 5,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "application": {
            "filter": "Zabbix server"
          },
          "functions": [
            {
              "$$hashKey": "object:431",
              "def": {
                "category": "Alias",
                "defaultParams": [],
                "name": "setAlias",
                "params": [
                  {
                    "name": "alias",
                    "type": "string"
                  }
                ]
              },
              "params": [
                "Zabbix busy housekeeper in %"
              ],
              "text": "setAlias(Zabbix busy housekeeper in %)"
            }
          ],
          "group": {
            "filter": "Zabbix servers"
          },
          "host": {
            "filter": "192.168.5.59_王总测试"
          },
          "item": {
            "filter": "Utilization of housekeeper internal processes, in %"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        },
        {
          "application": {
            "filter": "Zabbix server"
          },
          "functions": [
            {
              "$$hashKey": "object:445",
              "def": {
                "category": "Alias",
                "defaultParams": [],
                "name": "setAlias",
                "params": [
                  {
                    "name": "alias",
                    "type": "string"
                  }
                ]
              },
              "params": [
                "Zabbix busy trapper in %"
              ],
              "text": "setAlias(Zabbix busy trapper in %)"
            }
          ],
          "group": {
            "filter": "Zabbix servers"
          },
          "host": {
            "filter": "SVMGBZABPRD"
          },
          "item": {
            "filter": "Zabbix busy trapper processes, in %"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "C",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        },
        {
          "application": {
            "filter": "Zabbix server"
          },
          "functions": [
            {
              "$$hashKey": "object:459",
              "def": {
                "category": "Alias",
                "defaultParams": [],
                "name": "setAlias",
                "params": [
                  {
                    "name": "alias",
                    "type": "string"
                  }
                ]
              },
              "params": [
                "Zabbix busy alert manager in %"
              ],
              "text": "setAlias(Zabbix busy alert manager in %)"
            }
          ],
          "group": {
            "filter": "Zabbix servers"
          },
          "host": {
            "filter": "SVMGBZABPRD"
          },
          "item": {
            "filter": "Zabbix busy alert manager processes, in %"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "D",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        },
        {
          "application": {
            "filter": "Zabbix server"
          },
          "functions": [
            {
              "$$hashKey": "object:473",
              "def": {
                "category": "Alias",
                "defaultParams": [],
                "name": "setAlias",
                "params": [
                  {
                    "name": "alias",
                    "type": "string"
                  }
                ]
              },
              "params": [
                "Zabbix busy history syncer in %"
              ],
              "text": "setAlias(Zabbix busy history syncer in %)"
            }
          ],
          "group": {
            "filter": "Zabbix servers"
          },
          "host": {
            "filter": "SVMGBZABPRD"
          },
          "item": {
            "filter": "Zabbix busy history syncer processes, in %"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "E",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Zabbix Service Health",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {
        "CPU iowait time": "#F2C96D",
        "CPU system time": "#65C5DB",
        "CPU user time": "#E24D42"
      },
      "bars": false,
      "cacheTimeout": null,
      "dashLength": 10,
      "dashes": false,
      "datasource": "Zabbix",
      "editable": true,
      "error": false,
      "fill": 1,
      "fillGradient": 0,
      "grid": {},
      "gridPos": {
        "h": 8,
        "w": 6,
        "x": 18,
        "y": 26
      },
      "hiddenSeries": false,
      "id": 10,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 2,
      "links": [],
      "nullPointMode": "connected",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 5,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "application": {
            "filter": "CPU"
          },
          "functions": [],
          "group": {
            "filter": "$group"
          },
          "host": {
            "filter": "$host"
          },
          "item": {
            "filter": "Processor load (1 min average per core)"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "A",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        },
        {
          "application": {
            "filter": "CPU"
          },
          "functions": [],
          "group": {
            "filter": "Zabbix servers"
          },
          "host": {
            "filter": "SVMGBZABPRD"
          },
          "item": {
            "filter": "Processor load (15 min average per core)"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "B",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        },
        {
          "application": {
            "filter": "CPU"
          },
          "functions": [],
          "group": {
            "filter": "Zabbix servers"
          },
          "host": {
            "filter": "SVMGBZABPRD"
          },
          "item": {
            "filter": "Processor load (5 min average per core)"
          },
          "mode": 0,
          "options": {
            "showDisabledItems": false
          },
          "refId": "C",
          "resultFormat": "time_series",
          "table": {
            "skipEmptyValues": false
          },
          "triggers": {
            "acknowledged": 2,
            "count": true,
            "minSeverity": 3
          }
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Processor load",
      "tooltip": {
        "msResolution": true,
        "shared": true,
        "sort": 0,
        "value_type": "cumulative"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    }
  ],
  "refresh": "5m",
  "schemaVersion": 22,
  "style": "dark",
  "tags": [
    "Zabbix",
    "Version 4.4.0",
    "linux server"
  ],
  "templating": {
    "list": [
      {
        "allValue": null,
        "current": {
          "tags": [],
          "text": "Linux servers",
          "value": "Linux servers"
        },
        "datasource": "Zabbix",
        "definition": "*",
        "hide": 0,
        "includeAll": false,
        "index": -1,
        "label": "Group",
        "multi": false,
        "name": "group",
        "options": [
          {
            "$$hashKey": "object:848",
            "selected": false,
            "text": "Discovered hosts",
            "value": "Discovered hosts"
          },
          {
            "$$hashKey": "object:849",
            "selected": true,
            "text": "Linux servers",
            "value": "Linux servers"
          },
          {
            "$$hashKey": "object:850",
            "selected": false,
            "text": "Templates/Network devices",
            "value": "Templates/Network devices"
          },
          {
            "$$hashKey": "object:851",
            "selected": false,
            "text": "Templates/Operating systems",
            "value": "Templates/Operating systems"
          },
          {
            "$$hashKey": "object:852",
            "selected": false,
            "text": "Zabbix servers",
            "value": "Zabbix servers"
          }
        ],
        "query": "*",
        "refresh": 0,
        "regex": "",
        "skipUrlSync": false,
        "sort": 0,
        "tagValuesQuery": "",
        "tags": [],
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "allValue": null,
        "current": {
          "text": "192.168.4.10_研发编译",
          "value": "192.168.4.10_研发编译"
        },
        "datasource": "Zabbix",
        "definition": "$group.*",
        "hide": 0,
        "includeAll": false,
        "index": -1,
        "label": "Host",
        "multi": false,
        "name": "host",
        "options": [],
        "query": "$group.*",
        "refresh": 1,
        "regex": "",
        "skipUrlSync": false,
        "sort": 0,
        "tagValuesQuery": "",
        "tags": [],
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "allValue": null,
        "current": {
          "text": "All",
          "value": "$__all"
        },
        "datasource": "Zabbix",
        "definition": "$group.$host.*",
        "hide": 0,
        "includeAll": true,
        "index": -1,
        "label": "App",
        "multi": false,
        "name": "app",
        "options": [
          {
            "selected": true,
            "text": "All",
            "value": "$__all"
          },
          {
            "selected": false,
            "text": "CPU",
            "value": "CPU"
          },
          {
            "selected": false,
            "text": "Filesystems",
            "value": "Filesystems"
          },
          {
            "selected": false,
            "text": "General",
            "value": "General"
          },
          {
            "selected": false,
            "text": "IO",
            "value": "IO"
          },
          {
            "selected": false,
            "text": "Memory",
            "value": "Memory"
          },
          {
            "selected": false,
            "text": "Network interfaces",
            "value": "Network interfaces"
          },
          {
            "selected": false,
            "text": "OS",
            "value": "OS"
          },
          {
            "selected": false,
            "text": "Performance",
            "value": "Performance"
          },
          {
            "selected": false,
            "text": "Processes",
            "value": "Processes"
          },
          {
            "selected": false,
            "text": "Security",
            "value": "Security"
          },
          {
            "selected": false,
            "text": "Zabbix agent",
            "value": "Zabbix agent"
          }
        ],
        "query": "$group.$host.*",
        "refresh": 0,
        "regex": "",
        "skipUrlSync": false,
        "sort": 0,
        "tagValuesQuery": "",
        "tags": [],
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "allValue": null,
        "current": {
          "text": "All",
          "value": "$__all"
        },
        "datasource": "Zabbix",
        "definition": "Zabbix - application",
        "hide": 0,
        "includeAll": true,
        "index": -1,
        "label": "Network Interface",
        "multi": false,
        "name": "appnet",
        "options": [
          {
            "selected": false,
            "text": "All",
            "value": "$__all"
          },
          {
            "selected": true,
            "text": "Network interfaces",
            "value": "Network interfaces"
          }
        ],
        "query": {
          "$$hashKey": "object:778",
          "application": "/.*/",
          "group": "$group",
          "host": "$host",
          "item": "",
          "queryType": "application"
        },
        "refresh": 0,
        "regex": "/interface.*/",
        "skipUrlSync": false,
        "sort": 0,
        "tagValuesQuery": "",
        "tags": [],
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "allValue": null,
        "current": {
          "text": "All",
          "value": "$__all"
        },
        "datasource": "Zabbix",
        "definition": "$group.$host.$appnet.*",
        "hide": 0,
        "includeAll": true,
        "index": -1,
        "label": "",
        "multi": false,
        "name": "netif",
        "options": [
          {
            "selected": true,
            "text": "All",
            "value": "$__all"
          },
          {
            "selected": false,
            "text": "Incoming network traffic on eno16780032",
            "value": "Incoming network traffic on eno16780032"
          },
          {
            "selected": false,
            "text": "Outgoing network traffic on eno16780032",
            "value": "Outgoing network traffic on eno16780032"
          }
        ],
        "query": "$group.$host.$appnet.*",
        "refresh": 0,
        "regex": "/Bits (?:sent|received).*|^Incoming.*|^Outgoing.*/",
        "skipUrlSync": false,
        "sort": 0,
        "tagValuesQuery": "",
        "tags": [],
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      }
    ]
  },
  "time": {
    "from": "now-3h",
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
    ],
    "time_options": [
      "5m",
      "15m",
      "1h",
      "6h",
      "12h",
      "24h",
      "2d",
      "7d",
      "30d"
    ]
  },
  "timezone": "browser",
  "title": "Zabbix - Monitoring Linux servers v4.4.6",
  "uid": "UA28gl1Zz",
  "variables": {
    "list": []
  },
  "version": 6
}
```

## Reference

- [Zabbix - Monitoring Linux servers v4.4.0](https://grafana.com/grafana/dashboards/11209)
