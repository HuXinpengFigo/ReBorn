---
title: MacOS 安装配置LibTorch
date: 2021-05-13 15:17:12
index_img: /img/Xnip2021-05-13_15-18-56.png
banner_img: /img/Xnip2021-05-13_15-19-46.png
tags: [PyTorch,机器学习]
---

## 获取libtorch

获取libtorch有两种方法，一种参考PyTorch官网上的下载链接，一种为参考PyTorch官方文档的`wget`下载代码。<!-- more -->

### 手动下载

在官网的[Get Start](https://pytorch.org/get-started/locally/)里面选择C++/Java选项就会出现[下载链接](https://download.pytorch.org/libtorch/cpu/libtorch-macos-1.8.1.zip)。

![官网](/img/Xnip2021-05-13_15-06-07.png)

之后下载完成后，将下载下来的zip文件放在`~`目录下，等待后续操作。

### 使用命令代码下载

在[PyTorch官方文档](https://pytorch.org/cppdocs/installing.html)我们可以找到这样的一行命令：

```sh
wget https://download.pytorch.org/libtorch/nightly/cpu/libtorch-shared-with-deps-latest.zip
unzip libtorch-shared-with-deps-latest.zip
```

使用该命令可以下载libtorch，效果与手动下载相同。

### 在C++项目中配置LibTorch

该部分的代码主要参考[PyTorch官方文档](https://pytorch.org/cppdocs/installing.html)，但是文档的CMakeFile没有设定Torch的目录，我们需要自己设定`set(Torch_DIR /Users/figo/libtorch/share/cmake/Torch)`（此处地址为我电脑上的地址）。

CMakeFile完整文件：

```sh
cmake_minimum_required(VERSION 3.19)
project(torch_example)

set(Torch_DIR /Users/figo/libtorch/share/cmake/Torch)
find_package(Torch REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

add_executable(torch_example main.cpp)
target_link_libraries(torch_example "${TORCH_LIBRARIES}")
set_property(TARGET torch_example PROPERTY CXX_STANDARD 14)
```

测试用main文件：

```c++
#include <torch/torch.h>
#include <iostream>

int main() {
    torch::Tensor tensor = torch::rand({2, 3});
    std::cout << tensor << std::endl;
}
```

成功运行main文件并出现以下结果表示配置成功：

<img src="/img/Xnip2021-05-13_15-15-59.png" alt="main" style="zoom:50%;" />

```python
traced_script_module = torch.jit.trace(myNet,var)
traced_script_module.save("torch_script_eval.pt")
```



