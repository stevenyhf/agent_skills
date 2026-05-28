---
name: texttovideo

## 技能描述

将文本内容转换为视频文件的工具，支持自定义颜色方案、背景音乐和视频时长。使用 Python + FFmpeg + Pillow 实现文本渲染和视频编码。

## 触发场景

当用户需要：
- 将文本内容转换为视频
- 创建简单的文字视频
- 生成带背景音乐的视频
- 制作幻灯片开头或演示视频
- 快速创建教育内容视频

**触发词**：文本转视频、文字生成视频、文本生成视频、制作文字视频、创建文本视频

## 功能特性

- 支持多行文本自动换行
- 6种预设颜色方案（dark、light、blue、green、red、random）
- 支持自定义背景音乐（MP3、WAV、AAC）
- 高清视频输出（1920x1080）
- 自动文本对齐和尺寸调整
- 可自定义视频时长

## 必需依赖

- Python 3.6+
- FFmpeg
- Pillow（Python 库）

## 安装依赖

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ffmpeg python3-pip
pip3 install Pillow

# CentOS/RHEL
sudo yum install ffmpeg python3-pip
pip3 install Pillow

# macOS
brew install ffmpeg python3
pip3 install Pillow
```

## 使用方法

### 基础版本

```bash
python3 text_to_video.py "你的文本内容"

# 指定输出文件和时长
python3 text_to_video.py "欢迎使用" -o welcome.mp4 -d 10
```

### 高级版本（推荐）

```bash
# 基础使用
python3 text_to_video_advanced.py "你的文本内容"

# 指定颜色预设方案
python3 text_to_video_advanced.py "欢迎使用" -p blue -d 5

# 添加背景音乐
python3 text_to_video_advanced.py "带音乐的视频" -m background_music.mp3
```

## 命令行选项

- `-o, --output`：输出文件名（默认：output.mp4）
- `-d, --duration`：视频时长（秒）（默认：5）
- `-p, --preset`：颜色预设方案（dark、light、blue、green、red、random）
- `-m, --music`：背景音乐文件路径

## 颜色预设方案

- **dark**：深色背景，浅色文字
- **light**：浅色背景，深色文字
- **blue**：蓝色主题
- **green**：绿色主题
- **red**：红色主题
- **random**：随机配色

## 支持的音乐格式

- MP3
- WAV
- AAC

## 文件结构

```
texttovideo/
├── SKILL.md                    # 技能说明文档
├── text_to_video.py           # 基础版本（简单）
└── text_to_video_advanced.py  # 高级版本（推荐）
```

## 使用场景

- 创建幻灯片开头
- 生成教育内容视频
- 制作示例和演示
- 快速视频内容创建
- 社交媒体短视频制作

## 扩展功能建议

可根据需求扩展以下功能：
- 动画效果（渐显渐隐）
- 多帧文本切换
- 字体动态效果
- 文本语音合成（TTS）
- 自动视频编辑

## 技术实现

- 使用 Pillow 生成文本图像帧
- 使用 FFmpeg 将图像序列编码为视频
- 支持音频流合并
- 自动处理文本换行和居中对齐

## 许可证

MIT License

## 注意事项

- 确保 FFmpeg 已正确安装并添加到系统 PATH
- 视频时长建议在 1-60 秒之间
- 文本内容过长会自动调整字体大小
- 背景音乐文件需提前准备
