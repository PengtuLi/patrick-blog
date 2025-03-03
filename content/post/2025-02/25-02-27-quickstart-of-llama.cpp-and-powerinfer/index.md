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

## ggml

ggml是一个用 C 和 C++ 编写、专注于 Transformer 架构模型推理的机器学习库，ggml相当于c++版的pytroch。llama.cpp底层推理采用该库。
可以理解成它主要干了这个事`result = torch.matmul(matrix1, matrix2.T)`

关键概念：
- ggml_context: 一个装载各类对象 (如张量、计算图、其他数据) 的“容器”。

  对于ggml框架来说，无论要做什么（建立modle模型、建立计算图、还是创建承载计算结果的result）都需要先创建一个context作为容器，并将创建的结构体保存在context里（不包括实际数据本身，数据通过结构体里的指针索引）。

  ![ggml_context](ggml_context.png)

- ggml_cgraph: 计算图的表示，可以理解为将要传给后端的“计算执行顺序”。
- ggml_tensor：ggml版的tensor表示，与pytorch类似，数据存储在data指针变量里
- ggml_backend_t // the backend to perform the computation (CPU, CUDA, METAL)
- ggml_backend_buffer_t buffer; // the backend buffer to storage the tensors data

  在ggml框架中，一切数据（context、dataset、tensor...）都应该被存放在buffer中。之所以要用buffer进行集成，是为了便于实现多种后端（CPU、GPU）内存的统一管理。buffer是实现不同类型数据在多种类型后端上进行统一的接口对象。

  ggml_backend_buffer_t表示通过应后端backend通过分配的内存空间。需要注意的是，一个缓存可以存储多个张量数据。



要在CPU上完成上述矩阵乘法，步骤如下：
- 分配一个 ggml_context 对象来存储张量数据
- 分配张量并赋值
- 为矩阵乘法运算创建一个 ggml_cgraph
- 执行计算
- 获取计算结果
- 释放内存并退出

<details>

<summary>code</summary>

```c
#include "ggml.h"
#include "ggml-cpu.h"
#include <string.h>
#include <stdio.h>

int main(void) {
    // initialize data of matrices to perform matrix multiplication
    const int rows_A = 4, cols_A = 2;
    float matrix_A[rows_A * cols_A] = {
        2, 8,
        5, 1,
        4, 2,
        8, 6
    };
    const int rows_B = 3, cols_B = 2;
    float matrix_B[rows_B * cols_B] = {
        10, 5,
        9, 9,
        5, 4
    };

    // 1. Allocate `ggml_context` to store tensor data
    // Calculate the size needed to allocate
    size_t ctx_size = 0;
    ctx_size += rows_A * cols_A * ggml_type_size(GGML_TYPE_F32); // tensor a
    ctx_size += rows_B * cols_B * ggml_type_size(GGML_TYPE_F32); // tensor b
    ctx_size += rows_A * rows_B * ggml_type_size(GGML_TYPE_F32); // result
    ctx_size += 3 * ggml_tensor_overhead(); // metadata for 3 tensors
    ctx_size += ggml_graph_overhead(); // compute graph
    ctx_size += 1024; // some overhead (exact calculation omitted for simplicity)

    // Allocate `ggml_context` to store tensor data
    struct ggml_init_params params = {
        /*.mem_size =*/ ctx_size,
        /*.mem_buffer =*/ NULL,
        /*.no_alloc =*/ false,
    };
    struct ggml_context * ctx = ggml_init(params);

    // 2. Create tensors and set data
    struct ggml_tensor * tensor_a = ggml_new_tensor_2d(ctx, GGML_TYPE_F32, cols_A, rows_A);
    struct ggml_tensor * tensor_b = ggml_new_tensor_2d(ctx, GGML_TYPE_F32, cols_B, rows_B);
    memcpy(tensor_a->data, matrix_A, ggml_nbytes(tensor_a));
    memcpy(tensor_b->data, matrix_B, ggml_nbytes(tensor_b));


    // 3. Create a `ggml_cgraph` for mul_mat operation
    struct ggml_cgraph * gf = ggml_new_graph(ctx);

    // result = a*b^T
    // Pay attention: ggml_mul_mat(A, B) ==> B will be transposed internally
    // the result is transposed
    struct ggml_tensor * result = ggml_mul_mat(ctx, tensor_a, tensor_b);

    // Mark the "result" tensor to be computed
    ggml_build_forward_expand(gf, result);

    // 4. Run the computation
    int n_threads = 1; // Optional: number of threads to perform some operations with multi-threading
    ggml_graph_compute_with_ctx(ctx, gf, n_threads);

    // 5. Retrieve results (output tensors)
    float * result_data = (float *) result->data;
    printf("mul mat (%d x %d) (transposed result):\n[", (int) result->ne[0], (int) result->ne[1]);
    for (int j = 0; j < result->ne[1]/* rows */; j++) {
        if (j > 0) {
            printf("\n");
        }

        for (int i = 0; i < result->ne[0]/* cols */; i++) {
            printf(" %.2f", result_data[j * result->ne[0] + i]);
        }
    }
    printf(" ]\n");

    // 6. Free memory and exit
    ggml_free(ctx);
    return 0;
}

```

</details>

对于gpu上的执行该矩阵乘法：
https://github.com/ggerganov/ggml/blob/6c71d5a071d842118fb04c03c4b15116dff09621/examples/simple/simple-backend.cpp

## gguf格式

gguf特性：
- 方便添加新信息到模型中
- mmap兼容，能够直接将文件映射到内存中
- 量化兼容
- key-value structure netadata

图示：
![gguf](gguf.png)

增加、修改模型的metadata:
- https://github.com/ggml-org/llama.cpp/blob/master/gguf-py/gguf/scripts/gguf_set_metadata.py
- https://github.com/ggml-org/llama.cpp/blob/master/gguf-py/gguf/scripts/gguf_new_metadata.py

doc:
- https://github.com/ggml-org/ggml/blob/master/docs/gguf.md
- https://huggingface.co/docs/hub/gguf

## llama.cpp

### 参数转换

首先转换模型到gguf格式，参考
https://github.com/ggml-org/llama.cpp/blob/master/convert_hf_to_gguf.py#L4950

如 python convert_hf_to_gguf.py --outtype f16 --print-supported-models /mnt/models/opt-6.7b/

```md
模型转换流程:
1. 从HF模型路径下加载config.json文件，读取模型配置
hparams = Model.load_hparams(dir_model)

2. 依据 hparams 的模型架构，得到llama.cpp定义的对应的模型，如：GPTNeoXForCausalLM，并进行初始化
@Model.register("GPTNeoXForCausalLM")
class GPTNeoXModel(Model):

3. 然后调用llama.cpp定义的对应模型的接口，来完成模型转换
model_instance.set_gguf_parameters() # 将config.json中的参数配置写入到gguf文件的metadata中
model_instance.set_vocab()           # 进行词表转换，llama.cpp将词表数据也保存到了gguf文件中
```

如果模型没有定义不支持（如opt就不支持）
1. 先添加模型名称，参考
https://github.com/ggml-org/llama.cpp/blob/master/convert_hf_to_gguf_update.py

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

2. 还要在convert_hf_to_gguf.py添加该模型识别参数的类，类似`class GPTNeoXModel(Model)`

<mark>还没有太搞明白这里</mark>

### 运行

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
# e.g.:  ./build/bin/main -m /mnt/models/prosparse-llama-2-7b-gguf/prosparse-llama-2-7b.gguf -n 1024 -t 2 -p "write a story of sysu" --vram-budget 4--vram-budget 8
```

批量推理参考 https://github.com/SJTU-IPADS/PowerInfer/blob/main/examples/batched/README.md

powerinfer同时也支持量化模型，使用方式相同；
powerinfer暂时只支持ReLU/ReGLU/Squared，这三个激活函数，所以mistral, original llama,Qwen这些模型不支持,需要添加算子

## 参考
- https://www.bilibili.com/video/BV1Ez4y1w7fc 不错的视频解析llama.cpp
- https://www.bilibili.com/video/BV1N4wreWE8z 较为详细解析llama.cpp
- https://zhuanlan.zhihu.com/p/665027154
- https://zhuanlan.zhihu.com/p/691347732
- https://zhuanlan.zhihu.com/p/25774381094
- https://huggingface.co/blog/zh/introduction-to-ggml
- https://zhuanlan.zhihu.com/p/19968327329
