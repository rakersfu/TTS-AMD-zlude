# Qwen3‑TTS（适用于 AMD + ZLUDA）—— 源码安装指南（易读版）

本指南给出在 AMD 显卡 + ZLUDA 环境下，从零安装并运行 Qwen3‑TTS 的完整步骤。内容面向 Windows（但也包含常见的跨平台说明）。请按照顺序执行每一步。

---

## 重要说明 / 前提
- 强烈建议使用独立的 Python 环境，避免与其他项目（如 ChatTTS、ComfyUI）共用环境。
- 推荐 Python 版本：3.11（与 ZLUDA 兼容性更好）。
- 本方案通过让 PyTorch 使用“CUDA 版”并用 ZLUDA 替换/模拟 CUDA DLL，使 AMD GPU 可运行 CUDA 版 PyTorch。
- 操作前请备份重要环境与文件，替换 torch/lib 下 DLL 等操作有风险。

---

## 01 — 创建独立 Python 环境（必须）
Qwen3‑TTS 的依赖可能与其他项目冲突，请务必使用独立环境。

示例（Windows）：
```powershell
python -m venv qwen3-tts-venv
qwen3-tts-venv\Scripts\activate
```

示例（Linux / macOS）：
```bash
python -m venv qwen3-tts-venv
source qwen3-tts-venv/bin/activate
```

---

## 02 — 安装 CUDA 版 PyTorch（用于 ZLUDA）
虽然在 AMD 上运行，但需要安装“CUDA 版”的 PyTorch，然后通过 ZLUDA 替换或拦截 CUDA 调用。

示例（使用本地 whl 离线安装）：
```bash
pip install torch==2.1.2+cu118 torchvision==0.16.1 --no-index --find-links=./whl
```
要点：
- 必须安装 CUDA 版 torch（不要安装 ROCm 版）。
- 推荐版本：torch 2.1.2（兼容性较好）。
- 如果网络受限，可使用本地 whl 文件进行离线安装。

---

## 03 — 应用 ZLUDA Patch（关键步骤）
目标：将 PyTorch 所需的 CUDA 动态库替换为 ZLUDA 提供的实现，从而让 CUDA 二进制接口指向 ZLUDA，进而使用 AMD GPU。

步骤要点（概览）：
1. 获取 ZLUDA 的 DLL（或相应实现文件）。
2. 将 ZLUDA 的 DLL 复制并覆盖到你虚拟环境中 `torch/lib`（或 PyTorch 安装目录下的 `lib`）中对应的文件。
3. 需要替换或覆盖的库通常包括：cublas、cusparse、nvrtc、cufft、cufftw 等相关 DLL。
4. cuDNN 可视情况禁用或由 patch 处理（许多 ZLUDA 方案会处理 cuDNN 的兼容问题）。

重要提醒：
- 覆盖前请备份原来的 DLL。
- 确认替换成功：检查 `torch/lib` 下的文件是否已被覆盖（日期/大小/哈希）。
- 若不确定某些文件名，先查看 `torch/lib` 目录下现有文件，再比对替换。

---

## 04 — 克隆 Qwen3‑TTS 源码
获取最新源码并进入项目目录：

```bash
git clone https://github.com/QwenLM/Qwen3-TTS.git
cd Qwen3-TTS
```

确保你仍在上面创建并激活的虚拟环境中。

---

## 05 — 源码安装（开发模式）
推荐以“开发模式”安装，便于修改和调试：

```bash
pip install -e .
```

这会安装项目依赖（如 transformers、soundfile 等）并以可编辑模式安装源码。

---

## 06 — （可选）安装 FlashAttention 2
FlashAttention 可以在某些情形下降低显存占用，但在 ZLUDA 环境下不一定能带来加速效果，可视情况尝试安装。

基本安装：
```bash
pip install -U flash-attn --no-build-isolation
```

如果内存受限（例如 < 96 GB），可按 FlashAttention 的安装说明设置 `MAX_JOBS=4` 等编译参数。若安装失败，可以跳过该步骤，Qwen3‑TTS 仍能运行。

注意：FlashAttention 对 float16 / bfloat16 有更明显的效果。

---

## 07 — 通过 ZLUDA 运行 Qwen3‑TTS（必须）
运行时必须通过 ZLUDA 启动，使程序加载 ZLUDA 环境。

Windows 示例（假设使用 zluda.exe 启动器）：
```powershell
zluda.exe python your_script.py
```

关键点：
- 在代码里确保 ZLUDA Patch 在导入 torch 之前被应用，例如：
  ```python
  import zluda_patch  # 必须在 import torch 之前
  import torch
  ```
- 第一次运行会编译 kernel，可能较慢，请耐心等待编译完成。
- 若出现库加载或符号未找到的问题，请回到第 03 步检查 DLL 覆盖是否正确。

---

## 常见问题与排查建议
- 如果 PyTorch 仍未识别 GPU：检查是否安装了 CUDA 版的 torch（pip show torch 查看版本与标签），并确认 `torch/lib` 下的 DLL 已用 ZLUDA 覆盖。
- 如果出现 cuDNN 或特定 CUDA API 错误：确认 ZLUDA patch 是否完整覆盖了需要的库，或尝试禁用 cuDNN。
- 如果内核编译失败或报错：查看编译日志，确认系统上是否有必要的编译工具和头文件（视具体报错决定）。

---

## 快速命令汇总（可直接复制执行）
1. 创建并激活虚拟环境
```bash
python -m venv qwen3-tts-venv
# Windows
qwen3-tts-venv\Scripts\activate
# Linux/macOS
# source qwen3-tts-venv/bin/activate
```

2. 安装 CUDA 版 PyTorch（示例）
```bash
pip install torch==2.1.2+cu118 torchvision==0.16.1 --no-index --find-links=whl
```

3. 应用 ZLUDA Patch
- 手动复制 ZLUDA DLL 覆盖 `torch/lib` 下对应文件（cublas、cusparse、nvrtc、cufft、cufftw 等）

4. 克隆源码
```bash
git clone https://github.com/QwenLM/Qwen3-TTS.git
cd Qwen3-TTS
```

5. 源码安装
```bash
pip install -e .
```

6. 可选：安装 FlashAttention 2
```bash
pip install -U flash-attn --no-build-isolation
```

7. 通过 ZLUDA 运行
```bash
zluda.exe python your_script.py
```

---

## 我可以继续帮你做的事情
- 生成 Qwen3‑TTS + ZLUDA 的一键安装脚本（中/英文版本）。
- 生成适配 AMD 的 Qwen3‑TTS 声音克隆示例脚本。
- 生成自动检测 ZLUDA 是否生效的测试脚本（检测 PyTorch 是否使用 GPU、检测 zluda 提供的库是否被加载等）。
- 或者我可以把这个整理后的 README 直接提交到仓库（提交到 main 分支或打开 PR）。

请告诉我要继续哪一步（例如“生成一键安装脚本”或“提交到仓库”）。
