---
name: gemini
description: 用于一次性问答、摘要和生成的 Gemini CLI。
homepage: https://ai.google.dev/
metadata:
  {
    "brigade":
      {
        "emoji": "✨",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "安装 Gemini CLI（brew）",
            },
          ],
      },
  }
---

# Gemini CLI

以一次性模式使用 Gemini，通过位置参数传入提示词（避免交互模式）。

快速开始

- `gemini "Answer this question..."`
- `gemini --model <name> "Prompt..."`
- `gemini --output-format json "Return JSON"`

扩展

- 列出：`gemini --list-extensions`
- 管理：`gemini extensions <command>`

注意事项

- 如需认证，请先交互式运行一次 `gemini`，按登录流程操作。
- 为安全起见，避免使用 `--yolo`。