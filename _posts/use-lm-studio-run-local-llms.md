---
title: 使用 LM Studio 在 macOS 中本地运行大模型
date: 2024-11-21 12:26:10
tags:
 - LM Studio
 - Qwen
 - Llama
categories:
 - Note
---

## LM Studio

- [LM Studio 官网](https://lmstudio.ai/)  
- [LM Studio Github](https://github.com/lmstudio-ai)

> 本地运行大模型的工具中，LM Studio 和 Ollama 是最受欢迎的两款。在最近这一次的更新中，LM Studio 新增了对 MLX 的支持。
> Ref: [Mac跑大模型，首选LM Studio](https://medium.com/@huangyihe/mac%E8%B7%91%E5%A4%A7%E6%A8%A1%E5%9E%8B-%E9%A6%96%E9%80%89lm-studio-6f21dbe2912c)

选择 LM Studio 是因为自带 Chat 的 UI ，而且支持 MLX 。

不过 MLX 模型比较少，更新也慢。 

>MLX，是苹果公司开源的一个机器学习框架，专门为M系列芯片做了优化。  

<!--more-->

## 使用

下载安装后，点击左侧的搜索按钮可以下载模型

![LM Stuidio](https://m.nep.me/blog/post/lm-studio-get-model.png)

下载完成后点击顶部的 **Select a model to load** 就可以开始对话了。 

## 切换语言

LM Studio UI 支持多种语言，右下角点击设置的齿轮图标，在 General 中可以切换。

## 运行速度
模型占用的内存与实际模型的大小差不多。

输出速度：

**lmstudio-community/Qwen2.5-Coder-7B-Instruct-GGUF**  
```sh
42.12 tok/sec • 636 tokens • 0.20s to first token
```

**Qwen2.5-Coder-14B-Instruct-MLX-8bit**
```sh
15.42 tok/sec • 734 tokens • 1.19s to first token
```
这个运行起来比较卡顿，输出不是很连贯。

**Qwen2.5-Coder-14B-Instruct-MLX-8bit**
```sh
25.73 tok/sec • 758 tokens • 3.96s to first token
```

## 选择模型
**模型参数**（B 表示 Billion，十亿参数）是衡量模型复杂程度的关键指标之一。参数越大，模型的表现力和准确度通常越高，适合处理更复杂的任务，例如高质量的文本生成或对话理解。
本地使用常见的是 7/14B
- 7B ：参数较少，占用资源较低，运行速度较快，推荐基础问答或文案生成。
- 14B ：如果需要更高精度或复杂推理的任务，可以试试。

**量化版本**
量化是指将模型的权重数据从高精度（如 16bit 或 32bit）压缩到更低的精度（如 4bit 或 8bit），以降低模型的内存占用和计算需求。
- 4bit 模型：精度降低幅度较大，但大多数任务仍然能保持足够的准确度，同时运行速度更快、资源占用更少，适合对性能优化要求高的场景。
- 8bit 模型：相比 4bit，更接近原始模型的表现，适合需要更高质量输出的需求。

一般来说 4bit 的量化足够本地使用了，速度相对更快，占用资源更少。

## 问题
如果开启了输入法，使用 Enter 选词会立即提交，可以把提交改为 Ctrl + Enter 或是使用空格选词（