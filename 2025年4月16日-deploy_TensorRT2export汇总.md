---
title: 2023年11月16日 TensorRT推理汇总
date: 2025-03-16 17:30:04
category: AI & ML
subcategory: Deploy
tags:
  - deploy
  - tensorrt
---





去

### 

# 流程

**“ONNX → TensorRT Engine → 加载引擎文件并反序列化为可执行模型 → 分配输入输出 → 推理执行 → 拷出结果 → 后处理”**

> 1. ？反序列化为可执行模型
> 2. ？？ 创建执行上下文（IExecutionContext）
>
> 

## 1 Exe 导出

**加上 `--verbose`** 运行 `trtexec`



模型汇总

| 格式       | 文件扩展名         | 内容                                                  | 用途                                                         | 生成方式                                                     | 可直接推理？ |
| ---------- | ------------------ | ----------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------ |
| WTS        | `.wts`             | 网络结构各层的权重（文本或二进制）                    | 通常用于从 PyTorch 到 TensorRT 的自定义转换流程，如通过 C++ 的 API 加载 | 通常通过 PyTorch 脚本导出                                    | ❌            |
| TRT Engine | `.trt` / `.engine` | 已优化的推理引擎（含网络结构 + 权重 + TensorRT 优化） | TensorRT 的最终部署格式                                      | 使用 `trtexec` 或 TensorRT API 编译 `.onnx` 或 `.wts` 模型生成 | ✅            |









#  问题汇总

### ⚠️ 自定义层问题

#### 1 EyeLike动态形状

`EyeLike` 是 ONNX 中一个用于生成单位矩阵的操作。该错误说明：

- 你模型中的某个节点使用了 `EyeLike` 操作；
- 但这个操作的输入张量是动态维度（即维度在推理时不固定）；
- 而 TensorRT **不支持对动态维度使用 EyeLike**。

**EyeLike 操作**：这是一个自定义层或特殊层，在某些情况下，TensorRT 不支持动态形状。

**动态形状问题**：TensorRT 不支持动态形状的输入（`EyeLike` 操作的输入是动态形状），这导致了错误。

> 使用batch 大小不需要变化的minShapes时没用的！



### ⚠️ 动态输入尺寸问题：

#### 1 ONNX参数dynamic=True

你导出的 ONNX 模型使用了 **动态输入尺寸**（`dynamic=True`），导致 `trtexec` 在构建 TensorRT 引擎时需要你**明确地指定输入尺寸 profile**，否则就会报错。

解决：

> trtexec \
>   --onnx=best_dynamic.onnx \
>   --saveEngine=best.engine \
>   --fp16 \
>   --minShapes=images:1x3x640x640 \
>   --optShapes=images:8x3x640x640 \
>   --maxShapes=images:16x3x640x640

### ⚠️  量化问题

#### 1 INT64数据

你的 ONNX 模型中包含了一些 **`INT64` 类型的权重或常量**，而 TensorRT **原生只支持 `INT32` 及以下的整数类型**，因此它尝试**自动将 `INT64` 转换为 `INT32`**。

```
onnx2trt_utils.cpp:374: Your ONNX model has been generated with INT64 weights, while TensorRT does not natively support INT64. Attempting to cast down to INT32.

```



### ⚠️错误

[05/13/2025-15:25:59] [TRT] [E] 1: [defaultAllocator.cpp::nvinfer1::internal::DefaultAllocator::deallocate::61] Error Code 1: Cuda Runtime (invalid argument)











# success！

```
trtexec --onnx=F:\Pydata\DL_NEW\fuxian\ultralytics\runs\detect\wire-Ghost-C3ghost-GSCSP-CBAM3\weights\best_simplify_dynamic=False.onnx --saveEngine=best_int8.trt --int8 --verbose  --minShapes=images:1x3x640x640 --optShapes=images:8x3x640x640 --maxShapes=images:16x3x640x640trtexec ^
--onnx=F:\Pydata\DL_NEW\fuxian\ultralytics\runs\detect\wire-Ghost-C3ghost-GSCSP-CBAM3\weights\best_simplify_dynamic.onnx ^
--saveEngine=best_int8.trt ^
--int8 ^
--verbose ^
--minShapes=images:1x3x640x640 ^
--optShapes=images:8x3x640x640 ^
--maxShapes=images:16x3x640x640
```



![image-20250513135551397](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250513135551397.png)

