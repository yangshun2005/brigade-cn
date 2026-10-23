---
name: openai-whisper
description: 使用 Whisper CLI 进行本地语音转文字（无需 API 密钥）。
homepage: https://openai.com/research/whisper
metadata:
  {
    "brigade":
      {
        "emoji": "🎤",
        "requires": { "bins": ["whisper"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "openai-whisper",
              "bins": ["whisper"],
              "label": "安装 OpenAI Whisper（brew）",
            },
          ],
      },
  }
---

# Whisper（CLI）

使用 `whisper` 在本地转写音频。

快速开始

- `whisper /path/audio.mp3 --model medium --output_format txt --output_dir .`
- `whisper /path/audio.m4a --task translate --output_format srt`

说明

- 首次运行时模型会下载到 `~/.cache/whisper`。
- 此安装中 `--model` 默认为 `turbo`。
- 模型越小速度越快，越大准确度越高。