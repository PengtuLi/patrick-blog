---
title: "Ssh Port Forward"
author : "tutu"
description:
date: '2025-03-08'
lastmod: '2025-03-08'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['proxy']
---

转发前提：
- 端口转发是基于建立起来的SSH隧道，`隧道中断则端口转发终端`
- 只能在建立隧道时创建转发，不能为已有的隧道增加增加端口转发

<mark>正反向代理区别：正向代理服务端不知道真正的客户端，反向代理客户端不知道真正的服务端。可以把代理隧道理解为一个黑盒。</mark>

SSH隧道类型:
- 本地端口转发L（ssh服务作为正向代理）：本机侦听端口，本机访问被`转发到远程主机指定端口`
- 远程端口转发R（ssh服务作为反向代理）：远程侦听端口，远程访问被`转发到本地主机指定端口`
- 动态隧道模式

---

## 本地端口转发L

**eg:终端访问服务器**

```bash
ssh -L [本地地址:]本地端口:远程地址:远程端口 user@address -p 22
```

## 远程端口转发R

```bash
ssh -R [远程地址:]远程端口:本地地址:本地端口 user@address -p 22
```

## 动态端口转发D（与L区别：端口不固定 / 魔法）

仅需确定`本地地址:本地端口`，目标根据请求动态确定

```bash
ssh -R [本地地址:]本地端口 user@address -p 22
```
