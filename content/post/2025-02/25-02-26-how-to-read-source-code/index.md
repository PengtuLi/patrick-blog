---
title: "How to Read Source Code"
author : "tutu"
description:
date: '2025-02-26'
lastmod: '2025-02-26'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['thinking']
---

- learn from earlier unoptimized verion

一个技巧（learned from reading the source code of lamma.cpp）：阅读高度优化的代码以掌握底层概念是一种相当不理想的学习方法。建议通过在GitHub的`karpathy/llama2.c/run.c`的master分支中进行调试来代替。它是故意完全未优化的（即使是矩阵乘法也是最基本的“原始”实现。

同理，对于一个复杂的代码，我们可以找有没有人写出它的一个简要版本代码，或是从该库的早期版本学习。

- using the working exp and debugger to step into

In general, the starting point would be to get a working example and then use a debugger to step through the code.

This is absolutely what I would recommend. We don't have to try and understand libraries from their source code alone, we can use the running state, set breakpoints, and explore explore explore. I highly recommend using a debugger to see how llama.cpp or any open source code works!