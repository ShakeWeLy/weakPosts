---
title: 2026年6月11日 cpp learn
date: 2026-06-11 20:13:02
category: Engineering
subcategory: Backend
tags:
  - cpp
---

有了 GPU 支持，你可以进行 GPU 加速的计算：

### **1. 编译支持 CUDA 的 OpenCV**

bash

```
# 检查 OpenCV 是否已支持 CUDA
opencv_version=$(pkg-config --modversion opencv4)
echo "OpenCV version: $opencv_version"

# 检查 OpenCV 的 CUDA 支持（需要从源码编译带 CUDA 的版本）
opencv_config=$(pkg-config --cflags opencv4)
if echo "$opencv_config" | grep -q "CUDA"; then
    echo "OpenCV has CUDA support"
else
    echo "Current OpenCV does not have CUDA support (需要从源码编译)"
fi
```



## 2. 安装nvcc

```
# 只安装核心 CUDA 工具，不安装有问题的组件
sudo apt-get install -y --no-install-recommends cuda-compiler-12-4 cuda-libraries-dev-12-4

# 验证
/usr/local/cuda-12.4/bin/nvcc --version
```

