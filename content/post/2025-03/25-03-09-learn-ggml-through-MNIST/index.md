---
title: "Learn Ggml Through MNIST"
author : "tutu"
description:
date: '2025-03-09'
lastmod: '2025-03-09'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['deep learning']
---

为了深入了解ggml库，我们以官方的稍复杂的MINST示例作为参考: https://github.com/ggml-org/ggml/blob/master/examples/mnist/README.md

[一些简单的GGML介绍参考此前的博客quickstart-of-llama.cpp-and-powerinfer]({{<ref "/post/2025-02/25-02-27-quickstart-of-llama.cpp-and-powerinfer/index.md#ggml">}})

```bash
# 编译
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Debug -j 8

# 我们主要关注用fc的方式训练与推理

# train with pytorch
python3 mnist-train-fc.py mnist-fc-f32.gguf

# eval
build/bin/mnist-eval mnist-fc-f32.gguf data/MNIST/raw/t10k-images-idx3-ubyte data/MNIST/raw/t10k-labels-idx1-ubyte

# train
build/bin/mnist-train mnist-fc mnist-fc-f32.gguf data/MNIST/raw/train-images-idx3-ubyte data/MNIST/raw/train-labels-idx1-ubyte
```

