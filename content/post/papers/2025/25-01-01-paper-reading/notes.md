## note

<a id="2407.05858v2"></a>

### Fast On-device LLM Inference with NPUs

<mark>端侧推理，npu+cpu，量化int8</mark>

这篇是很经典的一篇利用NPU在端侧加速推理的一篇文章，已经被ASPLOS25录用。因为隐私计算的需求，以及手机性能的提高，端侧运行llm越来越可行，但是端到端延迟不符合要求。本文重点关注端侧的prefill阶段加速，在长序列的任务中，这部分计算延时占比可以达到80%以上。因为prefill阶段是计算密集型，且npu擅长整数并行运算且广泛存在于手机中，另外手机cpu/gpu并行计算能力相对于消费级gpu并行能力差，所以用npu加速prefill阶段的推理是很符合直觉的。

核心思想：在量化的模型下，在prefill阶段将尽可能多的整数计算移动到npu上执行，同时将浮点运算在cpu/gpu上执行。挑战：1.对于变长的prompt，npu只支持静态大小的操作，准备工作(构建优化计算图)耗时；2.npu计算与sota量化方法不匹配，常用的per-group量化无法在npu上执行，需要拆分矩阵运算；3.npu不擅长浮点运算，而attention和layernorm往往需要浮点计算以保持精度。

对于挑战1，提出了分块chunk与共享计算图，对于静态的计算图（如ffn），在不同chunk间共享，对于有序列位置依赖的计算图（如attention），不共享计算图；对于挑战2，提出了Shadow outlier execution，mllm.npu采取了per-tensor的量化方法以适应npu的结构，对于激活值中的离群值，将浮点部分移动到cpu上计算，最后合并结果。对于npu,cpu内存地址空间不共享，需要复制离群值对应的权重，这里采用了统计分析哪些channel容易出现离群值（hot,集中），预先复制，cold的权重按需从磁盘加载；对于挑战3，主要是解决如何协调npu/cpu间计算的问题，按chunk顺序调度然后重合计算肯定是不可行的，因为ATTENTION计算会变化，这里采用的是乱序执行，只要满足数据依赖关系，就可以执行，这里考虑的依赖包括chunk内部的依赖以及跨chunk的依赖。因为实验发现npu执行时间占推理的主要部分，所以是关键路径，我们要尽可能减少npu执行的停滞。所以对于具体调度来说，优先在cpu上调度subgraph(满足的后续依赖可以让npu执行最久)，在npu上则优先调度执行时间最长的subgraph。

<a id="2312.11514v3"></a>

### LLM in a flash

<mark>端侧推理(pc)，存储优化，激活稀疏，参数offload</mark>

端侧设备（个人pc）内存有限，需要offload模型参数到flash中，本文关注激活稀疏场景下从flash（nvme nand）加载参数到dram的通信优化。nvme特点是线程越多chunk越大带宽就越大，同时激活稀疏利用预测器可以只加载少量神经元。本文提出1.滑动窗口作为缓存的依据，因为神经元有一定agggregate的特性，所以保留窗口大小以前激活的神经元，这样可以减少传输的数据大小。2.bind neuron，增加chunk size，提高带宽。3.提出了一个在dram放置管理neurons的方法，不过感觉作用不大。

<a id="fast25-qin"></a>

### Mooncake

<mark>serving，</mark>

本文获得了fast25最佳论文奖，还是很牛的。本文的场景是巨大的Mass，该场景的特点是在满足SLO下最大化吞吐量。当然，这里有一个前置的配置就是说prefill与decode instance分离（普遍做法）。

在prefill阶段，kvcache前缀重用可以大大减小计算量，原始计算量$\text{flops}(n) = l \times (a n^2 d + b n d^2)$，假设n token中有前p个token被重用，则计算量变为原来的$(\frac{p}{n})^2$。假如kvcache不在prefill节点，则需要加载而只要传输时间小于减少部分计算时间，就有收益，经过计算llama3-70b + 1node(8*h800)只要20GB/s带宽（min NIC&h2d）即可，而这是很容易满足的。所以本文就提出核心思想`trading more storage for
less computation`，利用上一切可以用上的存储资源（DRAM,SSD,RDMA），尽可能提高prefix hit rate。挑战无非是：要提高hit rate,存储的kvcache就要很大，怎么存的问题；相对应的就是分布式环境不同设备传输kvcache的带宽问题；另一个就是基于serving的环境以及目标如何最优化调度的问题喽。

当一个请求到来时，调度器根据工作负载与kv cache状态调度到一个prefill与decode node pair。与传统的区别就是prefill前会进行Kvcache重用的加载，以及decode前会通过文件系统MOONCAKE Store传输到decode node(与prefill重叠)。

**MOONCAKE Store**，page block存储（page attention），hash id标记自己及prefix。

如果需要利用一切可以用上的存储资源，那么如何统一管理这个pool就是一个很麻烦的问题。本文提出Conductor作为全局调度器，