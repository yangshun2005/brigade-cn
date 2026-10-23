---
name: sag
description: ElevenLabs 文本转语音，提供 Mac 风格 say 的用户体验。
homepage: https://sag.sh
metadata:
  {
    "brigade":
      {
        "emoji": "🔊",
        "requires": { "bins": ["sag"], "env": ["ELEVENLABS_API_KEY"] },
        "primaryEnv": "ELEVENLABS_API_KEY",
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "steipete/tap/sag",
              "bins": ["sag"],
              "label": "安装 sag（brew）",
            },
          ],
      },
  }
---

# sag

使用 `sag` 调用 ElevenLabs TTS 并本地播放。

API 密钥（必需）

- `ELEVENLABS_API_KEY`（首选）
- CLI 也支持 `SAG_API_KEY`

快速开始

- `sag "Hello there"`
- `sag speak -v "Roger" "Hello"`
- `sag voices`
- `sag prompting`（针对具体模型的提示）

模型说明

- 默认：`eleven_v3`（富有表现力）
- 稳定：`eleven_multilingual_v2`
- 快速：`eleven_flash_v2_5`

发音与朗读规则

- 首选修正：重新拼写（如 "key-note"）、加连字符、调整大小写。
- 数字/单位/URL：`--normalize auto`（若损害人名则用 `off`）。
- 语言偏好：`--lang en|de|fr|...` 引导归一化。
- v3：不支持 SSML `<break>`；改用 `[pause]`、`[short pause]`、`[long pause]`。
- v2/v2.5：支持 SSML `<break time="1.5s" />`；`sag` 未暴露 `<phoneme>`。

v3 音频标签（放在行首）

- `[whispers]`、`[shouts]`、`[sings]`
- `[laughs]`、`[starts laughing]`、`[sighs]`、`[exhales]`
- `[sarcastic]`、`[curious]`、`[excited]`、`[crying]`、`[mischievously]`
- 示例：`sag "[whispers] keep this quiet. [short pause] ok?"`

默认语音

- `ELEVENLABS_VOICE_ID` 或 `SAG_VOICE_ID`

长内容输出前先确认语音和说话人。

## 聊天语音回复

当用户要求"语音"回复（如"疯狂科学家语音"、"用语音解释"）时，生成音频并发送：

```bash
# 生成音频文件
sag -v Rachel -o /tmp/voice-reply.mp3 "Your message here"

# 然后在回复中包含：
# MEDIA:/tmp/voice-reply.mp3
```

语音风格提示：

- 疯狂科学家：使用 `[excited]` 标签、戏剧性停顿 `[short pause]`、变化强度
- 平静：使用 `[whispers]` 或放慢语速
- 戏剧化：少量使用 `[sings]` 或 `[shouts]`

Rachel 的默认语音：`lj2rcrvANS3gaWWnczSX`（或直接用 `-v Rachel`）