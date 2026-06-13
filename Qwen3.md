🟦 Qwen3‑TTS 源码安装（适用于 AMD + ZLUDA）
下面是完整的、正确的、可直接执行的步骤。

01
创建独立 Python 环境（必须）
必做
Qwen3‑TTS 的依赖与其他项目冲突，因此必须使用独立环境。

推荐：python -m venv qwen3-tts-venv

不要与 ChatTTS 或 ComfyUI 共用环境

Python 版本建议 3.11（ZLUDA 最稳定）

创建后执行：qwen3-tts-venv\Scripts\activate

02
安装 CUDA 版 PyTorch（用于 ZLUDA）
Qwen3‑TTS 需要 PyTorch，但 AMD GPU 需要通过 ZLUDA 使用 CUDA 版。

示例：pip install torch==2.1.2+cu118 torchvision==0.16.1 --no-index --find-links=./whl

必须安装 CUDA 版 torch（不是 ROCm 版）

版本建议：torch 2.1.2（兼容性最好）

你可以使用本地 whl 离线安装

03
应用 ZLUDA Patch（关键步骤）
AMD GPU
将 CUDA DLL 替换为 ZLUDA DLL，使 PyTorch 能在 AMD GPU 上运行。

复制 ZLUDA DLL → 替换 torch/lib 下对应文件

替换 cublas、cusparse、nvrtc、cufft、cufftw

禁用 cuDNN（ZLUDA patch 已处理）

确保 torch/lib 下 DLL 被成功覆盖

04
克隆 Qwen3‑TTS 源码
从 GitHub 获取最新源码。

git clone https://github.com/QwenLM/Qwen3-TTS.git

cd Qwen3-TTS

确保你仍在 qwen3-tts-venv 环境中

05
源码安装（开发模式）
使用 pip install -e . 安装源码版本。

pip install -e .

会自动安装 transformers、soundfile 等依赖

适合你修改源码或调试

06
（可选）安装 FlashAttention 2
用于降低显存占用，但在 ZLUDA 下不一定能加速。

pip install -U flash-attn --no-build-isolation

如果内存 < 96GB：MAX_JOBS=4 pip install ...

仅在 float16/bfloat16 模式下生效

如果安装失败，可跳过

07
运行 Qwen3‑TTS（必须通过 ZLUDA）
确保模型在 AMD GPU 上运行。

zluda.exe python your_script.py

运行前确保 import zluda_patch 在 torch 之前

第一次运行会编译内核，较慢

🟩 你现在可以直接复制执行的命令（总结）
1. 创建环境
Code
python -m venv qwen3-tts-venv
qwen3-tts-venv\Scripts\activate
2. 安装 CUDA 版 PyTorch（用于 ZLUDA）
（你可以用自己的 whl）

Code
pip install torch==2.1.2+cu118 torchvision==0.16.1 --no-index --find-links=whl
3. 应用 ZLUDA Patch
（复制 DLL → 覆盖 torch/lib）

4. 克隆源码
Code
git clone https://github.com/QwenLM/Qwen3-TTS.git
cd Qwen3-TTS
5. 源码安装
Code
pip install -e .
6. 可选：安装 FlashAttention 2
Code
pip install -U flash-attn --no-build-isolation
7. 运行（必须通过 ZLUDA）
Code
zluda.exe python your_script.py
🟦 如果你愿意，我可以继续帮你：
生成 Qwen3‑TTS + ZLUDA 的一键安装脚本（英文/中文）

生成 Qwen3‑TTS 声音克隆示例脚本（AMD 适配版）

生成自动检测 ZLUDA 是否生效的测试脚本

你想继续哪一步。
