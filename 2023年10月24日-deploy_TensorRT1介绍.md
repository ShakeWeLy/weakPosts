---
title: 2023年10月24日 TensorRT1介绍
date: 2023-10-24 19:51:56
category: AI & ML
subcategory: Deploy
tags:
  - deploy
  - tensorrt
---

## 1 什么是TensorRT

### 1.1 **TensorRT介绍**

TensorRT是可以在**NVIDIA**各种**GPU硬件平台**下运行的一个**C++推理框架**。

我们利用Pytorch训练好的模型，可以转化为TensorRT的格式，然后利用TensorRT推理引擎去运行我们这个模型，从而提升这个模型在英伟达GPU上运行的速度。**速度提升**的比例是**比较可观**的。

### 1.2 TensorRT 工作流程

<img src="F:/.Hexo/MyBlog/source/_posts/2023年10月12日-TensorRT介绍/image-20250314191334384.png" alt="image-20250314191334384" style="zoom: 25%;" />



### 1.3 为什么能加速？

- **算子融合(层与张量融合)**：简单来说就是通过融合一些计算op或者去掉一些多余op来**减少数据流通次数以及显存的频繁使用来提速**，使用更少的cuda核心

  <img src="F:\.Hexo\MyBlog\source\_posts\2023年10月12日-TensorRT介绍\image-20250314190650887.png" alt="image-20250314190650887" style="zoom: 50%;" />

- **量化**：量化即IN8量化或者FP16以及TF32等不同于常规FP32精度的使用，这些精度可以显著提升模型执行速度并且不会保持原先模型的精度

- **内核自动调整**：根据不同的显卡构架、SM数量、内核频率等(例如1080TI和2080TI)，选择不同的优化策略以及计算方式，寻找最合适当前构架的计算方式`对特定显卡进行特定优化`为模型的每一层选择最优化的GPU 内核。这种自适应方法可确保模型充分利用 GPU 的计算能力。

  <img src="F:/.Hexo/MyBlog/source/_posts/2023年10月12日-TensorRT介绍/image-20231012202620295.png" alt="image-20231012202620295" style="zoom: 33%;" />

- **动态张量显存**/**动态Tensor 内存管理**：TensorRT 在推理过程中有效管理tensor 内存使用情况，减少内存开销并优化内存分配。这使得GPU 的内存使用效率更高。

- **多流执行**：使用CUDA中的stream技术，最大化实现并行操作

### 1.**4 TensorRT的加速效果怎么样**

加速效果取决于模型的类型和大小，也取决于我们所使用的显卡类型。

TensorRT所做的优化也是**基于GPU**进行优化，当然也是更喜欢那种一大块一大块的矩阵运算，尽量直通到底。因此对于通道数比较多的卷积层和反卷积层，优化力度是比较大的；

如果是比较繁多复杂的各种细小op操作**(例如reshape、gather、split等)，那么TensorRT的优化力度就没有那么夸张了。**

```
所以在转onnx的时候需要对模型进行修改，减少reshape等
```

### 1.5概念和使用方式

- 支持模型、数据类型、支持的IO、支持的功能

- TensorRT中context、engine、builder概念

- 环境搭建、运行环境

  **1.5 什么是算子**

  有时候自己会设计一些layer来满足任务需求，但是这些layer在使用Tensorrt优化时，TensorRT可能并不支持。

  这时候就需要自己写一个算子/**Plugin** 来实现 

  <img src="F:/.Hexo/MyBlog/source/_posts/2023年10月12日-TensorRT介绍/image-20250314191142137.png" alt="image-20250314191142137" style="zoom: 25%;" />

### 1.6 工作流程

- python端API
- C++端API
- 简单的demo

## 2 TensorRT完整流程

### 2.1 ONNX转TensorRT

- parser方式转换模型
- onnx2trt简单分析
- graphsurgeon修改onnx网络
- 一些注意点

### 2.2 框架内使用tensorrt

- torch_tensorrt介绍、使用
- torch_tensorrt源码解析

## 3 TensorRT进阶操作

### 3.1 如何debug

- 精度问题debug（nan、精度不匹配）
- 网络可视化（netron、trt-explorer）
- polygraphy（TensorRT官方提供的非常好的工具）

### 3.2 API搭建网络

- API搭建网络基本方式、权重提取，这部分强烈建议看TensorRTx
- explicit batch vs implicit batch

### 3.3 自定义插件plugin

- 如何写自定义plugin、Plugin中关键的API与注意点
- 自定义插件示例、集成plugin到trt中、参与trt序列化
- plugin的生命周期、资源管理
- dynamic-shape-plugin
- Plugin的FP16和INT8

### 3.4 常见疑难杂症

- 各种log中的报错

## 4 TensorRT最佳使用指南

### 4.1 如何正确使用trtexec

- trtexec基本用法
- 动态尺度、设置optimization profile以及注意点（min-opt-max）
- trtexec源码解析

### 4.2 提升性能

- 多steam（重叠计算和数据拷贝的时间，增加GPU利用率）
- 多context（多线程推理）
- 多optimization profile
- CUDA Graph（减少kernel launch时间）
- Timing Cache（减少build时间）
- Algorithm Selector

### 4.3 TensorRT转换相关

- 某些Layer选择的算法导致误差大，屏蔽掉该选择 tactic Source
- 更新权重refit功能（强化学习用的多）
- 构建期/运行期显存占用大（调整参数以及策略）

## 5 TensorRT量化

### PTQ

### QAT

- fake_
- QDQ 节点

### TensorRT-FP16

- fp16精度设置与使用
- fp16常见问题

### TensorRT-INT8

- int8使用
- PTQ校准集
- QAT量化

### 混合精度

- fp32和fp16
- fp16和int8

## 6 TensorRT拓展

### 6.1 TensorRT转换的几种方式

- torch2trt
- torchscript2trt
- fx2trt

## 7 TensorRT实战

### 6.1 检测模型转换

- SSD模型转TensorRT dynamic-shape
- 带有自定义DCN-OP的CenterNet转TensorRT

### 6.2 识别、分割、nlp类模型

- stable diffusion
- GPT

### 6.3 TensorRT+triton-inference-server线上/线下部署

- 基本部署教程
- 大规模TensorRT模型部署、多卡、模型调度
