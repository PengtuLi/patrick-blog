---
title: "Tmux Usage"
author : "tutu"
description:
date: '2025-01-21'
lastmod: '2025-01-21'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['tool']
---

## 速记

```raw
crtl-?                              # list all shortcuts

ctrl b [                            # 滑动窗口
tmux ls                             # 显示所有窗口
tmux new -s name                    # new session
tmux attach -t name                 # attach to session
crtl-b d, tmux detach               # detach


crtl-b s,w                          # list sessions,windows
crtl-b n,p                          # switch to next/prev window
crtl-b num                          # switch to specify window
Ctrl-b ,                            # rename window

crtl-b %,"                          # vsplit,hsplit
crtl-b x                            # close panel
crtl-b q                            # jump to a panel

```
