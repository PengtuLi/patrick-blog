---
title: "Block Scope in C++"
author : "tutu"
description:
date: '2025-03-04'
lastmod: '2025-03-04'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['program language']
---

在C++中，使用 {} 来包裹代码块是一种常见的做法，主要有以下作用和原因:

1. 定义变量的作用域，限制变量 i 的可见性，避免变量冲突；同时将变量生命周期限制在block内，离开块后自动清理变量。
2. 对相关逻辑进行分组，使代码更易读和维护。
