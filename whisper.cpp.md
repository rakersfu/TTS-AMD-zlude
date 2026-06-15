Whisper.cpp Vulkan + WebUI 使用指南
📦 环境准备
1. 安装必要工具
Visual Studio Build Tools  
下载并安装 Visual Studio Build Tools (visualstudio.microsoft.com in Bing)，勾选 MSVC 编译器 和 Windows SDK。

CMake  
安装 CMake，并确保添加到系统 PATH。

Ninja (可选)  
如果使用 Ninja 构建系统，安装 Ninja。

Git  
安装 Git for Windows，用于拉取源码。

Python 3.11+  
安装 Python，并确保 pip 可用。

Vulkan SDK  
下载并安装 LunarG Vulkan SDK，安装后会自动设置环境变量 VULKAN_SDK。
验证：

bat
echo %VULKAN_SDK%
dir %VULKAN_SDK%\Bin\glslc.exe
⚙️ 编译 Whisper.cpp
1. 拉取源码
bat
git clone https://github.com/ggerganov/whisper.cpp.git
cd whisper.cpp
2. 配置编译
使用 MSVC + Vulkan：

bat
cmake -B build -DGGML_VULKAN=1 -G "Visual Studio 17 2022" -A x64
3. 编译
bat
cmake --build build --config Release
编译完成后，主程序位于：

Code
build\bin\Release\whisper-cli.exe
📥 下载模型
进入 models 目录，下载所需模型，例如：

bat
cd models
curl -L -o ggml-base.en.bin https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-base.en.bin
curl -L -o ggml-large-v1.bin https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-large-v1.bin
可选模型：

ggml-tiny.en.bin （最快，精度低）

ggml-base.en.bin （平衡）

ggml-small.en.bin

ggml-medium.en.bin

ggml-large-v3.bin （最精确，显存需求大）

▶️ 命令行运行
转录示例音频
bat
.\build\bin\Release\whisper-cli.exe -m models\ggml-base.en.bin -f samples\jfk.wav
生成字幕文件
bat
.\build\bin\Release\whisper-cli.exe -m models\ggml-base.en.bin -f samples\jfk.wav --output-srt
🌐 WebUI 使用
1. 安装依赖
bat
pip install gradio
2. 运行 WebUI
在项目根目录创建 webui.py，内容如下：


import gradio as gr
import subprocess
import os
import shutil

WHISPER_BIN = r"F:/whisper.cpp/build/bin/Release/whisper-cli.exe"
MODEL_DIR = r"F:/whisper.cpp/models"
OUTPUT_DIR = r"F:/whisper.cpp/outputs"

os.makedirs(OUTPUT_DIR, exist_ok=True)

MODELS = [
    "ggml-tiny.en.bin",
    "ggml-base.en.bin",
    "ggml-small.en.bin",
    "ggml-medium.en.bin",
    "ggml-large-v3.bin"
]

def transcribe(audio_files, model_name, output_format):
    results = []
    model_path = os.path.join(MODEL_DIR, model_name)
    if not os.path.exists(model_path):
        return f"模型文件不存在: {model_path}"

    if not isinstance(audio_files, list):
        audio_files = [audio_files]

    for audio_file in audio_files:
        base_name = os.path.basename(audio_file)
        output_file = os.path.join(OUTPUT_DIR, base_name + "." + output_format)
        cmd = [WHISPER_BIN, "-m", model_path, "-f", audio_file]

        try:
            if output_format == "txt":
                # 捕获 stdout，避免 GBK 解码错误
                result = subprocess.run(
                    cmd,
                    capture_output=True,
                    text=True,
                    encoding="utf-8",
                    errors="ignore"
                )
                output_text = result.stdout if result.stdout else "转录失败，未捕获到输出"
                results.append(f"文件 {base_name} 转录结果:\n" + output_text)
            else:
                # srt/vtt 模式，生成文件
                if output_format == "srt":
                    cmd.append("--output-srt")
                elif output_format == "vtt":
                    cmd.append("--output-vtt")

                subprocess.run(cmd, check=True)

                # whisper-cli.exe 会在音频目录生成文件
                temp_output = audio_file + "." + output_format

                if os.path.exists(temp_output):
                    shutil.move(temp_output, output_file)
                    with open(output_file, "r", encoding="utf-8") as f:
                        results.append(f"文件 {base_name} 转录结果:\n" + f.read())
                else:
                    results.append(f"文件 {base_name} 转录失败，未生成输出文件。")

        except subprocess.CalledProcessError as e:
            results.append(f"文件 {base_name} 转录失败，错误信息: {e}")

    return "\n\n".join(results)

iface = gr.Interface(
    fn=transcribe,
    inputs=[
        gr.File(type="filepath", label="上传音频文件（可多选）", file_types=["audio"], file_count="multiple"),
        gr.Dropdown(MODELS, value="ggml-base.en.bin", label="选择模型"),
        gr.Radio(["txt", "srt", "vtt"], value="txt", label="输出格式")
    ],
    outputs="text",
    title="Whisper.cpp WebUI",
    description="上传音频，选择模型和输出格式，生成转录或字幕（支持批量处理）"
)

iface.launch(server_name="0.0.0.0", server_port=7860)


运行：

bat
python webui.py
浏览器访问：

Code
http://127.0.0.1:7860
上传音频 → 选择模型 → 选择输出格式 → 获取转录或字幕。

✅ 总结
安装工具链：VS Build Tools + CMake + Vulkan SDK + Python + Gradio。

编译 whisper.cpp，生成 whisper-cli.exe。

下载模型到 models 目录。

命令行或 WebUI 运行，生成转录或字幕。
