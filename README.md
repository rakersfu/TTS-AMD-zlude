# VoxCPM 使用说明

本指南基于 OpenBMB 的 VoxCPM 项目，说明如何在本地克隆、安装依赖并启动服务（推荐 Python 3.9+）。

> 参考仓库：https://github.com/OpenBMB/VoxCPM

---

## 1. 克隆项目代码
在命令行（CMD、PowerShell、Terminal）中执行：

```bash
git clone https://github.com/OpenBMB/VoxCPM.git
cd VoxCPM
```

---

## 2. 创建并激活虚拟环境（推荐）
建议使用 Python 3.9 及以上版本。

Windows:

```bash
python -m venv venv
# 激活 venv
.\venv\Scripts\activate
```

Linux / macOS:

```bash
python3 -m venv venv
# 激活 venv
source venv/bin/activate
```

激活虚拟环境后，命令行前缀通常会变成 `(venv)`，表示已进入虚拟环境。

---

## 3. 安装依赖（开发模式）
在已激活的虚拟环境中执行：

```bash
pip install -e .
```

说明：`-e`（editable）模式下，修改源码后无需重新安装即可生效，适合开发调试。

---

## 4. 启动 VoxCPM
运行以下命令启动应用，并指定端口（示例使用 8808）：

```bash
python app.py --port 8808
```

启动成功后，可以在浏览器访问：

http://127.0.0.1:8808

---

## 5. 可选：Windows 下的一键启动脚本（start_voxcpm.bat）
如果希望通过双击启动，可以在项目根目录创建一个批处理脚本 `start_voxcpm.bat`，示例内容如下：

```bat
@echo off
chcp 65001 >nul

:: Get current directory
set "BASEDIR=%~dp0"
cd /d "%BASEDIR%"

echo -----------------------------------------------------------------------
echo     * VoxCPM Starter (with venv check) *
echo -----------------------------------------------------------------------
echo.
echo Current script directory:
echo %BASEDIR%
echo.

:: Confirm path
set /p CONFIRM="Is the above path correct? (Y/N): "
if /I "%CONFIRM%" NEQ "Y" (
    echo Operation cancelled. Please check the path and run again.
    pause
    exit /b
)

echo.
echo Checking venv...
if not exist "%BASEDIR%venv\Scripts\python.exe" (
    echo ERROR: venv not found. Please create it first:
    echo     python -m venv venv
    pause
    exit /b
)
echo venv found.
echo.

:: Activate venv
call "%BASEDIR%venv\Scripts\activate.bat"

echo Current Python path:
where python
echo.

:: Start VoxCPM
python app.py --port 8808

pause

```

将以上内容保存为 `start_voxcpm.bat` 后，双击即可启动（会要求确认路径并检查 venv）。

---

## 常见注意事项

- 请确保 Python 与相关依赖的版本兼容，推荐 Python 3.9+。
- 如需使用 GPU 或特定后端，请参考 OpenBMB/VoxCPM 官方仓库的运行与依赖说明。
- 若在安装或运行过程中遇到权限或依赖问题，可尝试：
  - 在 Windows 上以管理员身份运行终端；
  - 使用 `python -m pip install --upgrade pip` 更新 pip 后重试；
  - 查看并安装系统层依赖（如 libsndfile、ffmpeg 等，视项目需求）。

---

## 总结
完整的本地运行流程：

1. 克隆项目
2. 创建并激活虚拟环境
3. 安装依赖（推荐开发模式 `pip install -e .`）
4. 启动应用并访问 http://127.0.0.1:8808

如果你希望我也把 `start_voxcpm.bat` 文件添加到仓库中，请告诉我，我可以一并提交。
