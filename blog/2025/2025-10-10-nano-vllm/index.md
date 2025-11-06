---
title: nano-vllm
authors: [OrionLi]
tags: ["llm", "code"]
---

## 简介

### 项目概述/核心价值

A lightweight vLLM implementation built from scratch.

- 1200loc,纯手搓
- 接近原生vllm速度
- 实现vllm的核心功能，例如前缀缓存，page attention,tensor并行，cuda graph
- 帮助快速上手复杂的vllm,把cuda kernel替换为了triton
- 不支持online推理

提供了一个bench的脚本，和一个qwen3 0.6b的模型文件

### 核心技术

### 技术栈

## 设计

### 架构

### 项目结构

### 数据流

## 核心工作流程

## 细节设计/关键模块

## one more thing

### stand higher

实现serve过程？

cuda kernel

other parallel

bench mark vs llm.swift

implement using minitorch?

### check

检验是否真正理解了 nano-vllm:

- 能够解释为什么 PagedAttention 比传统方式节省内存
- 能够描述一个 token 从输入到输出的完整生命周期
- 知道什么情况下会发生序列抢占，以及如何处理
- 能够解释张量并行中 All-Reduce 和 All-Gather 的作用
- 理解 CUDA Graph 为什么能加速，以及什么时候不适用
- 能够了解 FlashAttention 的优化动机和机理
- 里面的kernel是如何工作的

### 一些思考

- python对于package和module的使用以及引入方式，该项目用的是绝对包引用；以setuptool封装更容易使用,pip install -e .为edit mode
- python dataclass的使用
- init.py记得标明希望暴露的api-**ALL**,只需要暴露核心api
- 用户层面往往基于实现核心功能的base class继续封装，就算套层壳。例如class LLM,why?
  - 语义清晰 / 命名抽象,分离底层引擎与大模型业务
  - 可拓展，预留多拓展点，如stream chat
