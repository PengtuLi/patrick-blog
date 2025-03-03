---
title: "Linux Basic Usage"
author : "tutu"
description:
date: '2025-01-26'
lastmod: '2025-01-26'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['linux']
---

## 网络

```bash
#端口号查找
1.netstat
apt install net-tools
netstat -tuln ｜ grep xx
2.ss
ss -tulnp | grep xx

#网络状态
ifconfig
```

---

## 查询手册命令

```bash
man xxx
info xxx
```

---

## 压缩

```bash
1.tar
tar -czvf [压缩文件名].tar.gz [文件或目录名]  //打包并压缩// -c //wether to compress//
tar -xzvf [压缩文件名].tar.gz  解压.tar.gz文件
2.zip
zip xxx 
unzip xxx
```

---

## 磁盘

```bash
df -h # disk space human-readable
du -h //human-readable// -a //all-files// -d //depth//
```
