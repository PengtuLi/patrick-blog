---
title: "Neovim Ssh Remote"
author : "tutu"
description:
date: '2025-01-13'
lastmod: '2025-01-13'
image:
math: true
hidden: false
comments: true
draft: false
categories : ["tool"]
---

已经在本地配置好了neovim,现在希望的是有没有像vscode的remote-ssh一样的插件呢？
可以方便链接到远端服务器，同时不需要再把远端的vscode环境再手动配置一遍呢。

发现的一些方法：
- ssh连接远端服务器，配置nvim环境，再进行开发
- sshfs把远端文件系统同步到本地，但是这样需要解决一些包的安装问题，如pytorch等
- 像vscode remote插件一样

vscode的一些插件介绍：
- https://code.visualstudio.com/docs/remote/tunnels
- https://code.visualstudio.com/docs/remote/remote-overview 
- https://code.visualstudio.com/docs/devcontainers/containers

## Remote Nvim

https://github.com/amitds1997/remote-nvim.nvim/tree/main

我们计划使用这个插件，得到和vscode remote ssh插件类似的效果


>Why would I use this plugin instead of the usual ssh + nvim?
>This plugins provide some additional nice-to have features on top:
>
>- Automatically installs Neovim on remote
>- Does not mess with the global configuration and instead just writes everything to a single directory on remote
>- Can copy over your local Neovim configuration to remote
>- Allows easy re-connection to past sessions
>- Makes it easy to clean up remote machine changes once you are done
>- It launches Neovim server on the remote server and connects a UI to it locally.

