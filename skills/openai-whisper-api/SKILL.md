---
name: openai-whisper-api
description: 通过 OpenAI Audio Transcriptions API（Whisper）转写音频。
homepage: https://platform.openai.com/docs/guides/speech-to-text
metadata:
  {
    "brigade":
      {
        "emoji": "🌐",
        "requires": { "bins": ["curl"], "env": ["OPENAI_API_KEY"] },
        "primaryEnv": "OPENAI_API_KEY",
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "curl",
              "bins": ["curl"],
              "label": "安装 curl（brew）",
            },
          ],
      },
  }
---

# OpenAI Whisper API (curl)

通过 OpenAI 的 `/v1/audio/transcriptions` 端点转写音频文件。设置 `OPENAI_BASE_URL` 可使用兼容 OpenAI 的代理或本地网关。

## 快速开始

```bash
{baseDir}/scripts/transcribe.sh /path/to/audio.m4a
```

默认值：

- 模型：`whisper-1`
- 输出：`<input>.txt`

## 常用参数

```bash
{baseDir}/scripts/transcribe.sh /path/to/audio.ogg --model whisper-1 --out /tmp/transcript.txt
{baseDir}/scripts/transcribe.sh /path/to/audio.m4a --language en
{baseDir}/scripts/transcribe.sh /path/to/audio.m4a --prompt "Speaker names: Peter, Daniel"
{baseDir}/scripts/transcribe.sh /path/to/audio.m4a --json --out /tmp/transcript.json
```

## API 密钥

设置 `OPENAI_API_KEY`，或在当前生效的 Brigade 配置文件中配置（`$BRIGADE_CONFIG_PATH`，默认 `~/.brigade/brigade.json`）。可选设置 `OPENAI_BASE_URL`（例如 `http://127.0.0.1:51805/v1`）以使用兼容 OpenAI 的代理或本地网关：

```json5
{
  skills: {
    "openai-whisper-api": {
      apiKey: "OPENAI_KEY_HERE",
    },
  },
}
```