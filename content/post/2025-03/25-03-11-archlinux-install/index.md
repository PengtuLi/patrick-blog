---
title: "Archlinux Install with double system"
author : "tutu"
description:
date: '2025-03-11'
lastmod: '2025-03-11'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['linux']
---

计划在x86 windows主机上安装 archlinux 系统，以下是一些简明教程

## 准备工作

首先下载镜像 https://archlinux.org/download/

随后验证镜像

接下来烧写启动盘 https://github.com/pbatard/rufus 我们采样UEFI引导所以用GPT启动分区

随后把镜像复制进去

### 双系统安装

我这里想保留windows系统,所以需要空出磁盘空间，我这里把D分区删掉了，留给archlinux，这里我留出了270G的空间（nvme,和windows c盘一个物理硬盘）

{{<notice warning>}}

- 这里要记得关闭bitlock! archlinux不支持bitlock
- 禁用快速启动和安全启动（bios）
- 设置 Windows 时区偏移

```bash
reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f
```

{{</notice>}}

## 分区/格式化/挂载

接下来我们bios用启动盘进入。进入bash后，我们先连接网络。这里使用iwctl连接无线网络。

更新系统时间为UTC

```bash
hwclock --systohc --utc
timedatectl set-ntp true
```

接下来进行磁盘分区，

关于这里：

- /boot 1G
- /swap 4G
- /mnt 60G
- /home 200G左右

接下来我们格式化磁盘，分别格式化4个分区，我们这里选用经典的ext4文件系统。

最后我们挂载分区，这里要注意一下挂载的顺序，`lsblk`可以检查挂载情况。

```RAW
mount /dev/sda1 /mnt
mount /dev/sda2 /mnt/boot
swapon /dev/sda3
mount /dev/sda4 /mnt/home
```

## 安装系统

最后我们安装系统，这里用pacstrap脚本安装。（如果双系统还要安装ntfs工具）

我们最好改一下镜像源（reflactor工具）

## 配置系统

安装好后我们配置一下系统

- 生成 fstab 文件
- chroot 到新安装的系统
- 设置时间和时区
- 创建新用户，修改wheel
- 网络配置
- 安装微码
- 配置本地化选项
- 配置uefi引导

```raw
# uefi 双系统
pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub --recheck
# 为 Grub 启用 os-prober
echo "GRUB_DISABLE_OS_PROBER=false" >> /etc/default/grub
# 生成grub配置
grub-mkconfig -o /boot/grub/grub.cfg
```

结束！！！

## 安装桌面环境

refer: https://wiki.archlinux.org/title/GNOME

```BASH
pacman -S gnome gdm gnome-tweeks
# 开机自启
systemctl enable gdm
```

## 安装输入法

```RAW
# 安装输入法
pacman -S fcitx5-im
pacman -S fcitx5-chinese-addons

# 安装Input Method Pannel 扩展

# /etc/environment
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
INPUT_METHOD=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

## 参考

- Arch Linux 详细安装教程，萌新再也不怕了！「2023.10」 - 会心的文章 - 知乎
<https://zhuanlan.zhihu.com/p/596227524>
- https://www.cnblogs.com/NexusXian/p/17570030.html
- https://blog.linioi.com/posts/18/
- https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97