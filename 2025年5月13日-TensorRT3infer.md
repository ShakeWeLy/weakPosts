---
title: 2025年5月13日 TensorRT3infer
date: 2025-05-13 17:08:00
tags: TensorRT
---

## TensorRT 进行推理的流程



> ## ✅ 一般使用 TensorRT 进行推理的流程（7 步）
>
> ### 🔹 1. **准备模型（通常为 ONNX 格式）**
>
> - 训练好的模型需导出为 `ONNX` 格式。
> - 常见工具：
>   - PyTorch: `torch.onnx.export()`
>   - TensorFlow → ONNX: 使用 `tf2onnx` 或 `onnx-tf`
>   - HuggingFace 模型: 直接支持 ONNX 导出
>
> ------
>
> ### 🔹 2. **构建 TensorRT 引擎（`.engine` 文件）**
>
> 你可以使用：
>
> - **trtexec 命令行工具（推荐）**
>
>   ```
>   bash
>   
>   
>   复制编辑
>   trtexec --onnx=model.onnx --saveEngine=model.engine --fp16
>   ```
>
> - **TensorRT Python API（更灵活）**
>
>   ```
>   python复制编辑builder = trt.Builder(logger)
>   network = builder.create_network(...)
>   parser = trt.OnnxParser(network, logger)
>   parser.parse_from_file("model.onnx")
>   engine = builder.build_cuda_engine(network)
>   ```
>
> > 注意：此步骤可选，如果你已经有 `.engine` 文件，就无需重复构建。
>
> ------
>
> ### 🔹 3. **加载引擎文件并反序列化为可执行模型**
>
> ```
> cpp复制编辑std::ifstream engineFile("model.engine", std::ios::binary);
> IRuntime* runtime = createInferRuntime(logger);
> ICudaEngine* engine = runtime->deserializeCudaEngine(data, size, nullptr);
> ```
>
> ------
>
> ### 🔹 4. **创建执行上下文（IExecutionContext）**
>
> ```
> cpp
> 
> 
> 复制编辑
> IExecutionContext* context = engine->createExecutionContext();
> ```
>
> ------
>
> ### 🔹 5. **准备输入数据并分配 GPU 内存**
>
> - 你需要知道输入 tensor 的 shape（通常是 `[1, 3, H, W]`）
> - 分配输入/输出的 GPU 内存（`cudaMalloc`）
> - 把输入数据拷贝到 GPU（`cudaMemcpy`）
>
> ------
>
> ### 🔹 6. **执行推理（Inference）**
>
> - 使用 `enqueueV2`（异步）或 `executeV2`（同步）进行推理：
>
> ```
> cpp
> 
> 
> 复制编辑
> context->enqueueV2(buffers, stream, nullptr);
> ```
>
> - 注意：`buffers` 是输入/输出 tensor 指针数组。
>
> ------
>
> ### 🔹 7. **获取并处理输出结果**
>
> - 将输出结果从 GPU 拷贝到 Host：
>
> ```
> cpp
> 
> 
> 复制编辑
> cudaMemcpy(output_host, output_device, size, cudaMemcpyDeviceToHost);
> ```
>
> - 通常后处理包括：
>   - 分类任务：取最大值 `argmax`
>   - 检测任务：解析 bbox、类别置信度、NMS 等
>   - 分割任务：取最大通道作为 mask





