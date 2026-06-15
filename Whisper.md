# Whisper 在 Windows + AMD（ZLUDA）上的安装与使用指南

## 概览
本指南面向在 Windows 使用 AMD 显卡的用户，介绍在不支持原生 ROCm 的情况下，通过安装 CUDA 版 PyTorch 并使用 ZLUDA 替换 DLL，使 Whisper 能在 GPU 上运行的完整流程。并包含依赖安装、环境变量设置、测试以及常用用法示例。

---

## 一、环境准备

- Python：推荐 3.9–3.11（官方测试：3.9.9）。
- 建议使用虚拟环境（venv 或 conda）以避免污染系统环境。

### PyTorch（Windows + AMD 的推荐方案）
在 Windows + AMD 显卡下没有原生 ROCm 版 PyTorch。推荐安装 CUDA 版 PyTorch（如 cu118），然后使用 ZLUDA 替换 CUDA DLL。示例安装命令：

```bat
#如果需要代理
set HTTPS_PROXY=http://192.168.0.107.1:10808
pip install torch==2.7.0 torchvision==0.22.0 torchaudio==2.7.0 --index-url https://download.pytorch.org/whl/cu118
#出现问题后重新执行
pip install torch==2.7.0 torchvision==0.22.0 torchaudio==2.7.0 --index-url https://download.pytorch.org/whl/cu118 --force-reinstall
```

---

## 二、Whisper 安装

- 安装官方发布包：

```bat
pip install -U openai-whisper
```

- 或安装最新源码版本：

```bat
pip install git+https://github.com/openai/whisper.git
```

### 完整（WebUI）依赖示例 requirements.txt
以下为可能需要的依赖（若使用 WebUI 或扩展功能）：

```
numpy
numba
tqdm
more-itertools
setuptools

torch==2.7.0+cu118
torchvision==0.22.0+cu118
torchaudio==2.7.0+cu118

openai-whisper
tiktoken
ffmpeg

streamlit
fastapi
uvicorn
gradio

pydantic
requests
python-multipart

langchain
faiss-cpu
pdfplumber

triton>=2.0.0; platform_system=="Linux"
```

安装示例：

```bat
pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/cu118
```

---

## 三、系统依赖

### FFmpeg
在 Windows 下可用 Chocolatey 或 Scoop 安装：

```bat
choco install ffmpeg
```
或
```bat
scoop install ffmpeg
```

### Rust
若 tiktoken 等包没有预编译 wheel，可能需要安装 Rust：
- 到 Rust 官网下载安装器并安装。
- 确保 ~/.cargo/bin 在 PATH 中。

若遇到错误 `No module named 'setuptools_rust'`，可执行：

```bat
pip install setuptools-rust
```

---

## 四、AMD 显卡支持（使用 ZLUDA）
说明：ZLUDA 是一种在 AMD GPU 上模拟 CUDA 的方法，通过替换部分 CUDA DLL 以让 CUDA 版 PyTorch 在 AMD 上运行。

1. 下载 ZLUDA Windows 包（推荐使用 Gitee 镜像以获得更稳定的国内下载）。
2. 删除虚拟环境下旧的 CUDA DLL：
   - 位置：`venv\\Lib\\site-packages\\torch\\lib`（或 Python 安装目录下对应的 torch lib 目录）。
3. 复制并替换 ZLUDA 提供的 DLL，常见需要映射的文件（示例）：

```
cublas.dll → cublas64_11.dll
cublasLt.dll → cublasLt64_11.dll
cusparse.dll → cusparse64_11.dll
cufft.dll → cufft64_10.dll
cufftw.dll → cufftw64_10.dll
nvrtc.dll → nvrtc64_112_0.dll
```

4. 其他依赖（按 ZLUDA 包中的说明确认）：

- nvcuda.dll
- nvml.dll
- nccl.dll
- zluda_redirect.dll
- zluda_dump.dll

5. 设置环境变量（在命令行或系统环境变量中）：

```bat
set ZLUDA_VERBOSE=1
set HIP_VISIBLE_DEVICES=0
set CUDA_VISIBLE_DEVICES=0
```

6. 测试 GPU 是否可用：

```bat
zluda.exe -- python -c "import torch; print(torch.__version__); print(torch.version.cuda); print(torch.cuda.is_available())"
```

注意：
- 替换 DLL 和环境配置可能因 ZLUDA 版本、PyTorch 版本或 Windows 版本不同而需要调整；请以 ZLUDA 发布页的说明为准。 
- 建议保留原始 DLL 的备份，以便回滚。

---

## 五、使用 Whisper（示例）

### 命令行转录

- 基本转录：

```bat
whisper audio.wav --model turbo
```

- 指定语言（示例：日语）：

```bat
whisper japanese.wav --language Japanese
```

- 翻译成英文（示例）：

```bat
whisper japanese.wav --model medium --language Japanese --task translate
```

### Python 调用示例

```python
import whisper
model = whisper.load_model("turbo")
result = model.transcribe("audio.mp3")
print(result["text"])
```

---

## 六、常见问题与建议

- 如果在替换 DLL 后仍无法使用 GPU，请确认：
  - PyTorch 与 CUDA 版本是否匹配（例如：cu118 对应 torch 2.7+cu118）。
  - ZLUDA 版本是否支持当前的 CUDA 运行时名称（nvrtc 等）。
  - 是否正确设置了环境变量并重启终端/应用。 
- 若出现 tiktoken 编译/安装问题，优先尝试安装预编译 wheel，或安装 Rust 并确保 setuptools-rust 可用。

---

## 七、总结
在 Windows + AMD 显卡的环境下，推荐的流程为：

1. 创建并激活虚拟环境（venv 或 conda）。
2. 安装 CUDA 版 PyTorch（如 cu118）。
3. 安装 Whisper 及其它 Python 依赖（FFmpeg、tiktoken 等）。
4. 使用 ZLUDA 替换/映射必要的 CUDA DLL。
5. 配置环境变量并测试 torch.cuda.is_available()。
6. 使用 Whisper 进行转录或翻译。

如需我把某一部分展开（例如：按步骤列出具体 DLL 替换的命令、或提供 conda 环境的示例创建命令），我可以继续补充并更新文件.
