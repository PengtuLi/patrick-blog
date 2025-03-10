---
title: "Ssh Forward"
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

---
## SSH隧道类型

- 本地端口转发：本机侦听端口，访问`转发到远程主机指定端口`
- 远程端口转发：远程侦听端口，访问转发到本机主机指定端口
- 动态隧道模式
---
## 本地端口转发

**eg:终端访问服务器**
默认是22端口，不需要指出

场景:
> 远程云主机B1运行了一个服务，端口为3000，本地主机A1需要访问这个服务。

为啥需要本地端口转发呢？

> 云主机的防火墙默认只打开了22端口，如果需要访问3000端口的话，需要修改防火墙，需要配置允许访问的IP地址。麻烦，不安全。

何为本地端口转发：到本地端口的请求转发到目标主机的指定端口

```bash
ssh -L 本地地址:本地端口:目标地址:目标端口 user@address
```

---
## 远程端口转发

场景:
> 本地主机A1运行了一个服务，端口为3000，远程云主机B1需要访问这个服务。

需求：本地主机无公网ip（动态分配）,云主机无法访问本地主机

语法:
```bash
ssh -R 远程地址（可不填）:远程端口:目标地址:目标端口 user@address
```

---
## 动态端口转发（区别：端口不固定）(魔法)

场景:
> 远程云主机B1运行了多个服务，分别使用了不同端口，本地主机A1需要访问这些服务。

需求：
> 防火墙限制/为每个端口分别创建本地端口转发非常麻烦。

仅需确定`远程地址:远程端口`，目标根据请求动态确定

使用：
```bash
ssh -R localhost:remotePort:localhost:localPort user@address
```


---
## 代理原理

- 第一层：由你的流量发起段到代理软件的接入端，比如你的浏览器到你使用的Clash。
- 第二层：由代理软件到远端服务器，比如由你使用的Clash到某香港的服务器。
- 第三层：由远端服务器到目标服务器，比如由某香港的服务器到谷歌的服务器。

不管你使用什么代理软件，这些软件都会在本地监听一个端口来作为代理的接入端。比如Clash大多监听7890端口，v2rayNG默认监听10809端口。

通常情况下，使用Socks5代理或HTTP代理。区别在于HTTP代理只能够代理HTTP协议的流量，SOCKS5则支持代理几乎所有的TCP和UDP流量。所以在大部分场景下，代理软件监听的SOCKS5端口使用更多一些。

## 如何启用全局代理？

`通过TAP/TUN`

- TAP模式其实就是在设备上利用技术手段安装一个虚拟的”网卡”，从而使得系统所有的流量都能经过代理软件。
- 而TUN模式则和TAP模式异曲同工。TUN工作在更高的网络层，不需要虚拟的TAP设备，也能实现跟TAP类似的效果。不仅如此，TUN的性能也比TAP要好。目前，



---
## 测试代理

### 终端测试

```bash
curl -I google.hk
```
### Q：为什么终端curl无法访问而浏览器却可以访问谷歌?

使用clash中遇到的一个问题。在系统代理模式下，浏览器访问谷歌好好的，bash不管是ping还是curl都无法访问Google。

浏览器运行的端口是随机的（只要系统允许的端口）

这里找到一个原因可能是终端没有正确配置代理。在bash里用`env`命令也没有显示出proxy有关的环境变量。但是为什么设置里可以看到clash已经配置了http，htps，socks的代理。

查找发现：在macOS系统中，代理设置通常不会直接存储在环境变量中，而是通过系统偏好设置中的网络配置进行管理。在macOS系统中，终端访问网络时不会自动使用系统配置文件中的代理设置，这是因为macOS的网络代理设置和终端程序的网络访问是分开管理的。

解决方案：
```bash
export http_proxy="http://127.0.0.1/8888"
export https_proxy="http://127.0.0.1/8888"
export socks_proxy="socks://127.0.0.1/8885"
```

### Q:在终端运行程序下载文件，会不会使用系统代理呢?

>系统代理设置通常只影响那些明确支持系统代理设置的应用程序，比如一些浏览器。但是，并不是所有的应用程序都会使用系统代理设置。有些应用程序可能有自己的代理设置，或者可能根本不支持代理。

- **环境变量**：如果设置了环境变量，Python程序可能会使用这些代理。
- **代码中的代理设置**：如果Python代码中显式设置了代理，那么它将使用这些设置。
- **系统代理设置**：通常不会直接影响Python程序，除非通过环境变量。

---

## 配置代理
### 基于[v2rayA](https://v2raya.org/)
- 正常system环境
```bash
#realease下载https://github.com/v2rayA/v2rayA/releases
curl -LO https://github.com/v2rayA/v2rayA/releases/download/v2.2.5.8/installer_debian_x64_2.2.5.8.deb
#安装 deb 包
sudo apt install /path/download/installer_debian_xxx_vxxx.deb
#启动
sudo systemctl start v2raya.service
#开机启动
sudo systemctl enable v2raya.service
```
- docker环境
需要注意在docker容器内无法找到systemctl
同时在docker容器内（服务器租赁）无法启动透明代理，原因待排查
https://v2raya.org/docs/help/how-to-update/
https://github.com/v2fly/fhs-install-v2ray
### 基于ssh转发

待研究

ssj -j