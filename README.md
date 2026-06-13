🟥 AMD + ZLUDA（Windows）安装说明（中文整理版）
适用于：Stable Diffusion、ComfyUI、Z‑Image、Qwen3‑TTS、任意 PyTorch 程序
适配显卡：RDNA2（RX 6600 / 6700 / 6750 / 6800 / 6900 系列）

1. 安装 AMD 显卡驱动（Adrenalin）
使用任意 2026 年之后的 AMD Adrenalin 驱动即可。
无需特殊版本，无需 Pro 驱动。

2. 安装 HIP SDK（必须是 6.2.4）
⚠ 非常重要：只能安装 HIP SDK 6.2.4

HIP 7.x 会导致 ZLUDA 完全失效

HIP 7.x 还会触发杀毒软件误报

安装路径假设为：

Code
C:\Program Files\AMD\ROCm\6.2\
3. 安装 rocBLAS Tensile 库（关键步骤）
HIP SDK 默认 不包含 RDNA2 的 GEMM 优化内核。
如果没有 Tensile 内核，所有 matmul 都会报错：

Code
CUBLAS_STATUS_NOT_SUPPORTED
你必须安装与你显卡架构匹配的 Tensile 包：

GPU	架构
RX 6800 / 6900 / 6750 GRE	gfx1030
RX 6700 XT	gfx1031
RX 6600	gfx1032


步骤：

下载社区提供的 Tensile 包（如 likelovewant 的 rocBLAS 包）
文件名类似：

Code
rocm.gfx1031.for.hip.sdk.6.2.4.*.7z
解压到：

Code
C:\Program Files\AMD\ROCm\6.2\bin\rocblas\library\
⚠ 必须匹配 GPU 架构 + HIP 6.2.4。

4. 安装 Python 3.11
推荐版本：Python 3.11.9

⚠ Python 3.12+ 会导致部分 wheel 无法安装
⚠ 如果你系统默认 Python 不是 3.11，安装 ComfyUI 时要手动指定 Python 路径

5. 安装 ComfyUI + ZLUDA（推荐 patientx 版本）
最简单的方式是使用：

patientx/ComfyUI-Zluda fork

它会自动完成：

创建 venv

安装 ZLUDA‑patched 的 PyTorch 2.x + cu118

放置 ZLUDA 二进制文件到 ComfyUI-Zluda\zluda\

配置 comfyui-n.bat 启动器

自动应用 cuDNN wrapper（解决 VAE decode / conv 错误）

使用方法：

运行：

Code
install.bat
或：

Code
install-n.bat
ZLUDA 版本：3.9.5 nightly

ZLUDA 的编译缓存位置：

Code
%LOCALAPPDATA%\ZLUDA
⚠ 不要删除，否则每次都要重新编译 10 分钟。

6. Windows 系统注意事项（非常重要）
① Smart App Control（SAC）会阻止 ZLUDA
SAC 会拦截：

Code
nccl.dll
即使你添加 Defender 排除也无效。

解决：

Code
Windows 安全中心 → App & browser control → Smart App Control → Off
⚠ 一旦关闭 SAC，再打开需要 重置 Windows  
请谨慎操作。

② Windows Defender 误报（已知问题）
Defender 会误报：

Code
Trojan:Win32/Pomal!rfn
通常是针对：

Code
nccl.dll
解决：

给 ComfyUI-Zluda 文件夹添加排除

对误报文件点“允许”

7. 第一次启动 ComfyUI（ZLUDA 模式）
使用：

Code
comfyui-n.bat
注意：

① 第一次生成图像会卡 10–15 分钟
这是 ZLUDA 在为你的 GPU 编译内核。
只发生一次，之后就很快。

② RDNA2 推荐启动参数（已内置）
Code
--use-quad-cross-attention
--reserve-vram 0.9
--disable-async-offload
--disable-pinned-memory
环境变量：

Code
MIOPEN_FIND_MODE=2
这些参数可避免：

cuDNN 错误

conv_transpose1d 错误

VAE decode 错误

matmul 错误

③ comfyui-n.bat 每次启动会自动 git pull
优点：自动更新
缺点：上游更新可能导致兼容性问题

如果你想冻结版本：

Code
zluda.exe -- python main.py
🟩 总结（最简版）
步骤	内容
1	安装 AMD 驱动
2	安装 HIP SDK 6.2.4
3	安装对应 GPU 的 rocBLAS Tensile
4	安装 Python 3.11
5	安装 patientx/ComfyUI-Zluda
6	关闭 SAC + 添加 Defender 排除
7	使用 comfyui-n.bat 启动（首次编译需 10–15 分钟）


如果你需要，我可以继续为你整理：

Qwen3‑TTS（语音克隆）在 ZLUDA 上的中文安装说明

任意 PyTorch 程序在 ZLUDA 上运行的中文教程

AMD 6750 GRE 最佳配置指南
