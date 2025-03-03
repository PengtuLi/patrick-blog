---
title: "Hugging Face Download Model"
author : "tutu"
description:
date: '2024-12-18'
lastmod: '2024-12-18'
image:
math: true
hidden: false
comments: true
draft: false
categories : ["deep learning"]
---
## key

`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`

## 下载模型

- 从网页直接下载

- git clone下载仓库，不推荐，占空间无断点续传

- huggingface-cli

```bash

安装依赖

pip install -U huggingface_hub

镜像站点

export HF_ENDPOINT=https://hf-mirror.com

登陆

huggingface-cli login

下载模型/数据集

huggingface-cli download --repo-type model|dataset <repo_id> [filename] --local-dir <local_dir>

  

可选：hf-transfer

pip install -U hf-transfer

export HF_HUB_ENABLE_HF_TRANSFER=1

```

- AutoModel.from_pretrained自动下载，常见

- snap_download方法（完整库下载）

```python

from huggingface_hub import snapshot_download

  

repo_id = "meta-llama/Llama-2-7b-hf"  # 模型在huggingface上的名称

local_dir = "/home/model_zoo/LLM/llama2/Llama-2-7b-hf/"  # 本地模型存储的地址

local_dir_use_symlinks = False  # 本地模型使用文件保存，而非blob形式保存

token = "XXX"  # 在hugging face上生成的 access token

  

# 如果需要代理的话

proxies = {

    'http': 'XXXX',

    'https': 'XXXX',

}

  

snapshot_download(

    repo_id=repo_id,

    local_dir=local_dir,

    local_dir_use_symlinks=local_dir_use_symlinks,

    token=token,

    proxies=proxies

)

  

```

- hf_hub_download方法（可单文件）

```python

from huggingface_hub import hf_hub_download

hf_hub_download(

    repo_id="google/pegasus-xsum",

    filename="config.json",

    revision="4d33b01d79672f27f001f6abade33f22d993b151"

)

```

## Q: Facing SSL Error with Huggingface

- 好用

直接用镜像

`os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"`

- 不好用

```bash

降级requests和urllib3

pip install requests==2.27.1

pip install urllib3==1.25.11

os.environ["CURL_CA_BUNDLE"]=""

```

huggingface.co now has a bad SSL certificate, your lib internally tries to verify it and fails. By adding the env variable, you basically disabled the SSL verification.

- 换别的节点
