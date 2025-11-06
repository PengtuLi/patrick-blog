---
title: "Archlinux Install with double system"
authors: [OrionLi]
tags: ["linux"]
---

## 准备工作

- 首先下载镜像 https://archlinux.org/download/
- 随后验证镜像
- 接下来烧写启动盘 https://github.com/pbatard/rufus 我们采样UEFI引导所以用GPT启动分区
- 随后把镜像复制进去

## 分区/格式化/挂载

- 接下来我们bios用启动盘进入。进入bash后，我们先连接网络。这里使用iwctl连接无线网络。
- 更新系统时间为UTC
- 接下来进行磁盘分区，
  - /boot 1G
  - /swap equal to memory
  - /left space(btrfs)
- 最后我们挂载分区

## 安装系统

- 最后我们安装系统，这里用pacstrap脚本安装。
  - ucode

## 配置系统

- 生成 fstab 文件
- chroot 到新安装的系统
- 设置时间和时区
- 创建新用户到wheel组，修改wheel组权限，设置密码
- 配置本地化选项，locale
- 配置uefi引导

## 安装桌面环境

接下来正常开机进入系统

- desktop
  - display manager
  - window manager
  - xwayland like
- 中文字体
  - sudo pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei # 安装几个开源中文字体。一般装上文泉驿就能解决大多 wine 应用中文方块的问题
  - 有需要可以装微软的字体 yay -S ttf-ms-win11-auto-zh_cn
  - 配置中文字体 https://wiki.archlinuxcn.org/wiki/%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E6%9C%AC%E5%9C%B0%E5%8C%96
- 为图形界面配置中文 locale
  - ~/.xprofile 加入 export LANGUAGE=zh_CN:en_US
- 安装输入法
- 音频
- 蓝牙
- snapshot
  - grub util
- 显卡驱动
- 如果显示过小可以设置SDDM的dpi

## 其他

- 有需要可以更换内核
- 可以配置休眠到swap分区
- 功耗控制,超频啥的 https://wiki.archlinux.org/title/TLP_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

## 参考

- https://arch.icekylin.online/guide/advanced/beauty-1.html
