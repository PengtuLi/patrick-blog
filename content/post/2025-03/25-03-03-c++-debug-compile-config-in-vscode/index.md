---
title: "C++ Debug Compile Config in Vscode"
author : "tutu"
description:
date: '2025-03-03'
lastmod: '2025-03-03'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['tool']
---

## launch.json or task.json?

| 特性 | `tasks.json` | `launch.json` |
|------|--------------|---------------|
| **用途** | 定义和运行项目任务（如编译、打包、运行脚本等） | 配置调试环境（启动调试会话、设置断点、单步执行等） |
| **功能** | 执行命令或脚本，无需调试功能 | 启动调试器，支持调试功能（如断点、变量查看等） |
| **关键字段** | `label`, `type`, `command`, `args`, `options` | `name`, `type`, `request`, `program`, `args`, `env` |
| **使用场景** | 执行常规任务（如编译代码、运行测试等） | 调试程序（查找问题、优化代码等） |
| **文件位置** | `.vscode/tasks.json` | `.vscode/launch.json` |

**核心区别**：  
- `tasks.json` 是用来运行任务的，适合执行命令和脚本。  
- `launch.json` 是用来调试程序的，适合启动调试会话。  

两者在开发中通常配合使用，先通过 `tasks.json` 构建项目，再通过 `launch.json` 调试运行结果。

## debug

file:`${workspace}/.vscode/launch.json`

```json
- cmd
cuda_visible_device=0 ./build/bin/llama-simple -m /mnt/models/Llama-2-7b-hf/Llama-2-7B-hf-F16.gguf "Hello my name is"

- launch.json
{
    "version": "2.0.0",
    "configurations": [
        {
            "name": "llama-simple",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/bin/llama-simple",
            "args": ["-m", "/mnt/models/Llama-2-7b-hf/Llama-2-7B-hf-F16.gguf", "Hello my name is"],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",//调试器的当前工作目录，它是代码中使用的任何相对路径的基础文件夹。如果省略，默认为${workspaceFolder}
            "environment": [
                {
                    "name": "CUDA_VISIBLE_DEVICES",
                    "value": "0"
                }
            ],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "Set Disassembly Flavor to Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```