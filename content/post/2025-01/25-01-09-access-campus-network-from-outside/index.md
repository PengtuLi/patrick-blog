---
title: "Access Campus Network From Outside"
author : "tutu"
description:
date: '2025-01-09'
lastmod: '2025-01-09'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['proxy']
---

寒假回家，需要访问校园网内的服务器怎么办呢？

目前可行的方法是通过学校提供的easyconnect软件进行流量转发,使用的应该是ssh隧道，但是每次使用都要登陆以及随机验证码验证，十分不方便，而且休眠后连接就会终端。

个人设备：
- 内网服务器一台，无公网ip，配置了ssh服务
- 内网主机一台，无公网ip，配置了todesk远程桌面
- 外网笔记本一台，无公网ip
- 阿里云服务器一台，有公网ip

经过调研，可行的方案：
1. ss