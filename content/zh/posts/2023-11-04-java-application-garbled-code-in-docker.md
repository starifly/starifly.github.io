---
keywords:
- 
title: "docker部署java应用乱码问题"
date: 2023-11-04T09:42:36+08:00
lastmod: 2023-11-04T09:42:36+08:00
description: ""
draft: false
author: starifly
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
tags: [java,docker]
categories: [java,docker]
slug: java-application-garbled-code-in-docker
img:
---

在dockerfile中添加`ENV LANG=en_US.utf8`
