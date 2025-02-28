---
title: "Quickstart of Llama.cpp and Powerinfer"
author : "tutu"
description:
date: '2025-02-27'
lastmod: '2025-02-27'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['deep learning']
---

## llama.cpp

### 使用

首先转换模型到gguf格式，参考
https://github.com/ggml-org/llama.cpp/blob/master/convert_hf_to_gguf.py#L4950

如 python convert_hf_to_gguf.py --outtype f16 --print-supported-models /mnt/models/opt-6.7b/

如果模型不支持，先添加模型，参考https://github.com/ggml-org/llama.cpp/blob/master/convert_hf_to_gguf_update.py

如添加opt则
{"name": "opt-6.7b","tokt": TOKENIZER_TYPE.BPE, "repo": "https://huggingface.co/facebook/opt-6.7b", }
添加完后需要运行convert_hf_to_gguf_update

```
# Instructions:
#
# - Add a new model to the "models" list
# - Run the script with your huggingface token:
#
#   python3 convert_hf_to_gguf_update.py <huggingface_token>
#
# - The convert_hf_to_gguf.py script will have had its get_vocab_base_pre() function updated
# - Update llama.cpp with the new pre-tokenizer if necessary
```
还没有太搞明白这里

获取guuf格式模型后，编译llama.cpp，把cuda后端参数设置为on。

编译完成后运行测试,一个简单的生成测试为例，https://github.com/ggml-org/llama.cpp/blob/master/examples/simple/README.md

### 结构分析

代码结构：

## powerinfer

### 使用

编译和llama.cpp差不多,目前编译会有很多warning,不知道有没有问题。

模型权重格式有区别，*.powerinfer.gguf，包括模型权重与预测器权重
```raw
.
├── *.powerinfer.gguf (Unquantized PowerInfer model)
├── *.q4.powerinfer.gguf (INT4 quantized PowerInfer model, if available)
├── activation (Profiled activation statistics for fine-grained FFN offloading)
│   ├── activation_x.pt (Profiled activation statistics for layer x)
│   └── ...
├── *.[q4].powerinfer.gguf.generated.gpuidx (Generated GPU index at runtime for corresponding model)
```

转换模型参数from Original Model Weights + Predictor Weights至.powerinfer.gguf格式，参考
```raw
python convert.py --outfile ./ReluLLaMA-70B-PowerInfer-GGUF/llama-70b-relu.powerinfer.gguf ./SparseLLM/ReluLLaMA-70B ./PowerInfer/ReluLLaMA-70B-Predictor
```

推理参考
```raw
./build/bin/main -m /PATH/TO/MODEL -n $output_token_count -t $thread_num -p $prompt --vram-budget $vram_gb
# e.g.: ./build/bin/main -m ./ReluLLaMA-7B-PowerInfer-GGUF/llama-7b-relu.powerinfer.gguf -n 128 -t 8 -p "Once upon a time" --vram-budget 8
```
批量推理参考 https://github.com/SJTU-IPADS/PowerInfer/blob/main/examples/batched/README.md

powerinfer同时也支持量化模型，使用方式相同；
powerinfer暂时只支持ReLU/ReGLU/Squared，这三个激活函数，所以mistral, original llama,Qwen这些模型不支持,需要添加算子

## 参考
- https://www.bilibili.com/video/BV1Ez4y1w7fc
- https://zhuanlan.zhihu.com/p/665027154