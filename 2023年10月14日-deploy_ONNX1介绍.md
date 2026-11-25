---
title: ONNX介绍
date: 2023-08-01 14:07:53
category: AI & ML
subcategory: Deploy
tags:
  - deploy
  - onnx
---



## **0x1. ONNX介绍**

神经网络交换（Open Neural Network Exchange）

何种训练框架训练模型（比如TensorFlow/Pytorch/OneFlow/Paddle），在训练完毕后你都可以将这些框架的模型统一转换为ONNX这种**统一的格式**进行存储。注意ONNX文件不仅仅存储了神经网络模型的权重，同时也存储了模型的结构信息以及网络中每一层的输入输出和一些其它的辅助信息。



## **0x2. ProtoBuf简介**

ONNX使用的是Protobuf这个序列化数据结构去存储神经网络的权重信息。

Protobuf是一种轻便高效的**结构化数据存储格式**，

`*.proto`后缀文件  `onnx.proto`就是ONNX格式文件



## **0x3. ONNX格式分析**

ONNX 模型加载后返回的是一个 `ModelProto` 对象，其结构如下：

- **ModelProto**
  - `ir_version`：IR（中间表示）版本
  - `producer_name/producer_version`：创建模型的工具名与版本
  - `graph`：GraphProto 对象，定义了网络结构

- **GraphProto**
  - `node[]`：计算节点数组，每个是一个 NodeProto（例如Conv、Relu等操作）
  - `input[]`：模型的输入信息，类型为 ValueInfoProto
  - `output[]`：模型的输出信息，类型为 ValueInfoProto
  - `initializer[]`：模型的参数（如权重），类型为 TensorProto



## **0x4. onnx.runtime**



## **0x5. onnx-simplifier**

**简化 ONNX 模型的计算图结构**，以提升模型推理效率与部署兼容性。主要作用包括：

- **图结构优化**：将复杂的计算图变得更简单，例如合并连续的操作（如多个 `Transpose`）、常量折叠、节点冗余移除等。
- **提升兼容性**：有些深度学习推理引擎在面对复杂计算图结构时可能无法正确解析，简化后的模型更容易被不同的推理框架支持。
- **提高推理性能**：在保持模型精度不变的前提下，减少冗余计算，提高实际部署中的推理速度。
- **如动态输入**：

## 0x6.ONNX量化

### 6.1 量化概述

pass

### 6.2 量化方式

ONNXRuntime 支持两种模型量化方式：

1. 动态量化：

   对于动态量化，缩放因子（Scale）和零点（Zero Point）是在推理时计算的，并且特定用于每次激活

   因此它们更准确，但引入了额外的计算开销

2. 静态量化：

   对于静态量化，它们使用校准数据集离线计算

   所有激活都具有相同的缩放因子（Scale）和零点（Zero Point）

####    6.3 量化数据类型

- ONNXRuntime 支持两种量化数据类型：

- - Int8 (QuantType.QInt8): 有符号 8 bit 整型

  - UInt8 (QuantType.QUInt8): 无符号 8 bit 整型

  - 数据类型选择：

  - - 结合激活和权重，数据格式可以是（activation：uint8，weight：uint8），（activation：uint8，weight：int8）等。
    - 这里使用 U8U8 作为 （activation：uint8， weight：uint8） 的简写，U8S8 作为 （activation：uint8， weight：int8） 和 S8U8， S8S8 作为其他两种格式的简写。
    - CPU 上的 OnnxRuntime Quantization 可以运行 U8U8，U8S8 和 S8S8。
    - 具有 QDQ 格式的 S8S8 是性能和准确性的默认设置，它应该是第一选择。
    - 只有在精度下降很多的情况下，才能尝试U8U8。
    - 题。

#### 6.4 量化格式

#### **1. Tensor-Oriented（QuantFormat.QDQ）**

- 该格式使用 **`QuantizeLinear` 和 `DequantizeLinear`** 两个算子显式地插入在模型计算图中，表示张量的量化与反量化过程。

- 每个量化张量都会通过如下模式处理：

  ```
  text
  
  
  复制编辑
  original_tensor → QuantizeLinear → Quantized Tensor → DequantizeLinear → dequantized_tensor
  ```

- 优点：

  - 表现形式直观，清晰显示了量化与反量化操作；
  - 适合用于分析与调试；
  - 在动态量化（dynamic quantization）和量化感知训练（QAT）中使用广泛；

- 缺点：

  - 推理图中**节点数较多**，不如 QOperator 紧凑；
  - 某些后端或引擎（如 TensorRT）不完全支持。

------

#### **2. Operator-Oriented（QuantFormat.QOperator）**

- 该格式直接使用专门的量化运算符，例如：

  - `QLinearConv`
  - `QLinearMatMul`
  - `MatMulInteger`
  - `ConvInteger`

- 每个运算符自带量化参数（如 scale 和 zero point），无需额外的 `QuantizeLinear` / `DequantizeLinear` 包裹。

- 优点：

  - 推理效率高，节点数更少；
  - ONNX Runtime 在 CPU 上有良好支持；

  缺点：

  - 构建图更复杂，调试不如 QDQ 格式直观；
  - 某些工具链或后端对该格式支持有限（如部分硬件 NPU）

#### 6.6 比较

- 量化前后的模型文件大小如下表所示：

| 模型     | 大小   |
| -------- | ------ |
| 原始模型 | 16.3MB |
| 优化模型 | 16.1MB |
| 动态量化 | 4.1MB  |
| 静态量化 | 4.1MB  |