---
title: "Vllm Spec Decoding Impl"
author : "tutu"
description:
date: '2025-03-14'
lastmod: '2025-03-14'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['deep learning']
---

本文主要是分析vllm推测解码实现的部分

## vllm 推测解码伪代码分析

```raw
// implement sd class
// Only top-1 proposal and scoring are implemented. Tree-attention is left as future work.
class SpecDecodeWorker：

    // Perform speculative decoding on the input batch.
    function execute_model(execute_model_req):
        
        
        track finished requests
        check if all speculation should be disabled
        determine if it's prefill phase or other cases where speculation is not used
        broadcast necessary info to non-driver workers
        if speculation is disabled:
            run models without speculation
            handle hidden states and prepare for next step
            serialize output if logprobs are disabled
            return the result
        else:
            run speculative decoding step
            generate proposals using draft worker
            score proposals using scoring worker
            verify tokens to determine accepted ones
            create output sampler list based on accepted tokens
            track sequences with bonus tokens
            return the sampler output list

```