## llm infer

### Fast On-device LLM Inference with NPUs

`D. Xu et al., “Fast On-device LLM Inference with NPUs,” Dec. 15, 2024, arXiv: arXiv:2407.05858. doi: 10.48550/arXiv.2407.05858.`

<mark>端侧推理，npu+cpu，量化int8</mark>

这篇是很经典的一篇利用NPU在端侧加速推理的一篇文章，已经被ASPLOS25录用。因为隐私计算的需求，以及手机性能的提高，端侧运行llm越来越可行，但是端到端延迟不符合要求。本文重点关注端侧的prefill阶段加速，在长序列的任务中，这部分计算延时占比可以达到80%以上。因为prefill阶段是计算密集型，且npu擅长整数并行运算且广泛存在于手机中，另外手机cpu/gpu并行计算能力相对于消费级gpu并行能力差，所以用npu加速prefill阶段的推理是很符合直觉的。

核心思想：在量化的模型下，在prefill阶段将尽可能多的整数计算移动到npu上执行，同时将浮点运算在cpu/gpu上执行。挑战：1.对于变长的prompt，npu只支持静态大小的操作，准备工作(构建优化计算图)耗时；2.npu计算与sota量化方法不匹配，常用的per-group量化无法在npu上执行，需要拆分矩阵运算；3.npu不擅长浮点运算，而attention和layernorm往往需要浮点计算以保持精度。

对于挑战1，提出了分块chunk与共享计算图，对于静态的计算图（如ffn），在不同chunk间共享，对于有序列位置依赖的计算图（如attention），不共享计算图；对于挑战2，提出了Shadow outlier execution，mllm.npu采取了per-tensor的量化方法以适应npu的结构，对于激活值中的离群值，将浮点部分移动到cpu上计算，最后合并结果。对于npu,cpu内存地址空间不共享，需要复制离群值对应的权重，这里采用了统计分析哪些channel容易出现离群值（hot,集中），预先复制，cold的权重按需从磁盘加载；对于挑战3，主要是解决如何协调npu/cpu间计算的问题，按chunk顺序调度然后重合计算肯定是不可行的，因为ATTENTION计算会变化，这里采用的是乱序执行，只要满足数据依赖关系，就可以执行，这里考虑的依赖包括chunk内部的依赖以及跨chunk的依赖。因为实验发现npu执行时间占推理的主要部分，所以是关键路径，我们要尽可能减少npu执行的停滞。所以对于具体调度来说，优先在cpu上调度subgraph(满足的后续依赖可以让npu执行最久)，在npu上则优先调度执行时间最长的subgraph。

### LLM in a flash: Efficient Large Language Model Inference with Limited Memory

`K. Alizadeh et al., “LLM in a flash: Efficient Large Language Model Inference with Limited Memory,” Jul. 30, 2024, arXiv: arXiv:2312.11514. doi: 10.48550/arXiv.2312.11514.`

<mark>端侧推理(pc)，存储优化，激活稀疏，参数offload</mark>

端侧设备（个人pc）内存有限，需要offload模型参数到flash中，本文关注激活稀疏场景下从flash（nvme nand）加载参数到dram的通信优化。nvme特点是线程越多chunk越大带宽就越大，同时激活稀疏利用预测器可以只加载少量神经元。本文提出1.滑动窗口作为缓存的依据，因为神经元有一定agggregate的特性，所以保留窗口大小以前激活的神经元，这样可以减少传输的数据大小。2.bind neuron，增加chunk size，提高带宽。3.提出了一个在dram放置管理neurons的方法，不过感觉作用不大。
