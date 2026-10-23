---
name: summarize
description: 从 URL、播客和本地文件中总结或提取文本/转录稿（“转写这个 YouTube/视频”的绝佳备选方案）。
homepage: https://summarize.sh
metadata:
  {
    "brigade":
      {
        "emoji": "🧾",
        "requires": { "bins": ["summarize"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "steipete/tap/summarize",
              "bins": ["summarize"],
              "label": "安装 summarize（brew）",
            },
          ],
      },
  }
---

# Summarize

快速总结 URL、本地文件和 YouTube 链接的 CLI。

## 何时使用（触发短语）

用户提出以下任一请求时，立即使用此技能：

- “使用 summarize.sh”
- “这个链接/视频讲了什么？”
- “总结这个 URL/文章”
- “转写这个 YouTube/视频”（尽力提取转录稿；无需 `yt-dlp`）

## 快速开始

```bash
summarize "https://example.com" --model google/gemini-3-flash-preview
summarize "/path/to/file.pdf" --model google/gemini-3-flash-preview
summarize "https://youtu.be/dQw4w9WgXcQ" --youtube auto
```

## YouTube：摘要 vs 转录稿

尽力提取转录稿（仅限 URL）：

```bash
summarize "https://youtu.be/dQw4w9WgXcQ" --youtube auto --extract-only
```

如果用户要求转录稿但内容过长，先返回简短摘要，再询问要展开哪个部分/时间段。

## 模型 + 密钥

为所选提供商设置 API 密钥：

- OpenAI：`OPENAI_API_KEY`
- Anthropic：`ANTHROPIC_API_KEY`
- xAI：`XAI_API_KEY`
- Google：`GEMINI_API_KEY`（别名：`GOOGLE_GENERATIVE_AI_API_KEY`、`GOOGLE_API_KEY`）

未设置时默认模型为 `google/gemini-3-flash-preview`。

## 常用参数

- `--length short|medium|long|xl|xxl|<chars>`
- `--max-output-tokens <count>`
- `--extract-only`（仅限 URL）
- `--json`（机器可读）
- `--firecrawl auto|off|always`（备用提取）
- `--youtube auto`（设置 `APIFY_API_TOKEN` 时使用 Apify 备用方案）

## 配置

可选配置文件：`~/.summarize/config.json`

```json
{ "model": "openai/gpt-5.2" }
```

可选服务：

- `FIRECRAWL_API_KEY` 用于被屏蔽的站点
- `APIFY_API_TOKEN` 用于 YouTube 备用方案