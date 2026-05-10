# AutoDL 防系统盘爆满：环境配置完整流程（通用版）

## 0. 目标

AutoDL 默认很多东西都会安装到系统盘 `/root`。

但系统盘通常只有：

```text
30GB
```

很容易被：

- conda 环境
- pip 缓存
- HuggingFace 模型
- torch 编译缓存
- CUDA extension

占满。

因此目标是：

> 将 conda、环境、缓存、模型全部放到数据盘 `/root/autodl-tmp`。

适用于：

- 深度学习项目
- CUDA 项目
- Robotics 项目
- LLM / VLM 项目
- 任何需要 conda / pip / torch 的项目

---

# 1. AutoDL 的磁盘结构

AutoDL 通常有两种盘：

| 路径 | 作用 | 特点 | 大小 |
|---|---|---|---|
| `/` | 系统盘 | 小，容易爆 | 30GB |
| `/root/autodl-tmp` | 数据盘 | 大，用来放环境和项目 | 50GB(可付费扩容) |

默认很多东西会写入：

```text
/root
```

例如：

```text
/root/miniconda3
/root/.cache
/root/.local
```

所以：

# 核心原则

> 所有环境和缓存都应该放到 `/root/autodl-tmp`

---

# 2. 安装 Miniconda 到数据盘

## 2.1 进入数据盘

```bash
cd /root/autodl-tmp
```

---

## 2.2 下载 Miniconda

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

---

## 2.3 开始安装

```bash
bash Miniconda3-latest-Linux-x86_64.sh
```

---

## 2.4 最关键的一步（seriously 我就踩了这个坑）

安装过程中会出现：

```text
[/root/miniconda3] >>>
```

这里不要直接回车。

输入：

```text
/root/autodl-tmp/miniconda3
```

意思是：

> 把 conda 安装到数据盘。

---

## 2.5 初始化

出现：

```text
Proceed with initialization? [yes|no]
```

输入：

```text
yes
```

---

# 3. 激活数据盘 conda

执行：

```bash
source ~/.bashrc
source /root/autodl-tmp/miniconda3/etc/profile.d/conda.sh
conda activate base
```

然后检查：

```bash
which conda
```

正确结果应该是：

```text
/root/autodl-tmp/miniconda3/bin/conda
```

---

# ⚠️ 非常重要

如果你看到：

```text
/root/miniconda3/bin/conda
```

说明：

# 你正在使用系统盘 conda

修复方法：

```bash
source /root/autodl-tmp/miniconda3/etc/profile.d/conda.sh
conda activate base
```

---

# 4. 配置 conda 缓存到数据盘

## 4.1 设置 classic solver

```bash
conda config --set solver classic
```

新版 libmamba 有时会兼容性不好（可能会遇到报错，切换成 classic 就可以跑）。

---

## 4.2 设置 conda package 缓存路径

```bash
conda config --add pkgs_dirs /root/autodl-tmp/miniconda3/pkgs
```

---

## 4.3 清理旧缓存（新实例貌似没啥用，但有比没有好）

```bash
conda clean -a -y
```

---

## 4.4 验证

```bash
conda config --show pkgs_dirs
```

应包含：

```text
/root/autodl-tmp/miniconda3/pkgs
```

---

# 5. 配置 pip / HuggingFace / torch 等缓存

## 5.1 创建缓存目录

```bash
mkdir -p /root/autodl-tmp/.cache/pip
mkdir -p /root/autodl-tmp/.cache/huggingface
mkdir -p /root/autodl-tmp/.cache/torch_extensions
```

---

## 5.2 设置环境变量

```bash
export PIP_CACHE_DIR=/root/autodl-tmp/.cache/pip

export HF_HOME=/root/autodl-tmp/.cache/huggingface

export TORCH_EXTENSIONS_DIR=/root/autodl-tmp/.cache/torch_extensions
```

---

## 5.3 写入 bashrc（永久生效）

```bash
echo 'export PIP_CACHE_DIR=/root/autodl-tmp/.cache/pip' >> ~/.bashrc

echo 'export HF_HOME=/root/autodl-tmp/.cache/huggingface' >> ~/.bashrc

echo 'export TORCH_EXTENSIONS_DIR=/root/autodl-tmp/.cache/torch_extensions' >> ~/.bashrc
```

---

## 5.4 生效

```bash
source ~/.bashrc
```

---

# 6. 检查 GPU 和 CUDA

## 检查 GPU

```bash
nvidia-smi
```

如果正常，会看到 GPU 型号（如果你是无卡模式是看不到的，别慌）。

例如：

```text
RTX 4090
```

---

## 检查 CUDA

```bash
nvcc --version
```

---

# 7. 配置 CUDA_HOME

很多 CUDA 项目会编译 CUDA extension。

例如：

- PyTorch3D
- Grounded-SAM-2
- flash-attn
- detectron2

所以需要告诉系统：

> CUDA 安装在哪里。

设置：

```bash
export CUDA_HOME=/usr/local/cuda
```

写入 bashrc：

```bash
echo 'export CUDA_HOME=/usr/local/cuda' >> ~/.bashrc
```

生效：

```bash
source ~/.bashrc
```

检查：

```bash
echo $CUDA_HOME
```

应输出：

```text
/usr/local/cuda
```

---

# 8. 创建 conda 环境

举例（要根据你的项目环境要求来配置）：

```bash
conda create -n myenv python=3.10 -y
```

激活：

```bash
conda activate myenv
```

检查：

```bash
which python
```

应该类似：

```text
/root/autodl-tmp/miniconda3/envs/myenv/bin/python
```

说明环境在数据盘。

---

# 9. 安装项目依赖

通常有三种方式：

## requirements.txt

```bash
pip install -r requirements.txt
```

---

## environment.yml

```bash
conda env create -f environment.yml
```

---

## install.sh / install_env.sh

```bash
bash install.sh
```

或者：

```bash
bash install_env.sh
```

---

# 10. 下载模型权重（optional）

很多项目需要下载模型。

建议：

- 下载到数据盘
- 不要放 `/root`

例如：

```bash
mkdir -p /root/autodl-tmp/checkpoints
```

---

# 11. 标准检查流程

## 检查 conda

```bash
which conda
```

应是：

```text
/root/autodl-tmp/miniconda3/bin/conda
```

---

## 检查环境

```bash
conda info --envs
```

---

## 检查 python

```bash
which python
```

---

## 检查 torch

```bash
python -c "import torch; print(torch.cuda.is_available())"
```

---

## 检查 GPU

```bash
nvidia-smi
```

---

# 12. 常见错误总结（速查）

---

## 12.1 系统盘爆满

原因：

```text
/root/miniconda3
```

解决：

# conda 必须安装到数据盘

---

## 12.2 which conda 错误

看到：

```text
/root/miniconda3/bin/conda
```

说明：

# 又切回系统盘 conda 了

修复：

```bash
source /root/autodl-tmp/miniconda3/etc/profile.d/conda.sh
conda activate base
```

---

## 12.3 CondaToSNonInteractiveError

新版 conda 第一次使用官方源时需要接受 ToS。

解决：

```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main

conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
```

---

## 12.4 ChecksumMismatchError

原因：

- 下载损坏
- cache 混乱
- 用了旧 conda

解决：

```bash
conda clean -a -y
```

然后重新安装。

---

## 12.5 Killed

原因：

- 无卡模式内存不足
- conda solver 爆内存

解决：

```bash
export CONDA_SOLVER_THREADS=1
export CONDA_REPODATA_THREADS=1
```

或者直接换 GPU 实例。

---

## 12.6 CUDA extension 编译失败

常见报错：

```text
_C not defined
CUDA_HOME is not set
```

解决：

```bash
export CUDA_HOME=/usr/local/cuda
```

---

# 13. 推荐最终目录结构

```text
/root/autodl-tmp/
│
├── miniconda3/
│
├── project/
│
├── checkpoints/
│
└── .cache/
    ├── pip/
    ├── huggingface/
    └── torch_extensions/
```

---

# 14. Last but not least

真正容易出问题的不是 Python 本身，而是：

- conda 路径
- CUDA 版本
- cache 路径
- torch extension 编译
- 系统盘和数据盘混用

只要：

```text
所有环境和缓存都在 /root/autodl-tmp
```

后面的开发会稳定很多（希望吧）。
