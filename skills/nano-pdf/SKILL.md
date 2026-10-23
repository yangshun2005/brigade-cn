---
name: nano-pdf
description: 使用 nano-pdf CLI 通过自然语言指令编辑 PDF。
homepage: https://pypi.org/project/nano-pdf/
metadata:
  {
    "brigade":
      {
        "emoji": "📄",
        "requires": { "bins": ["nano-pdf"] },
        "install":
          [
            {
              "id": "uv",
              "kind": "uv",
              "package": "nano-pdf",
              "bins": ["nano-pdf"],
              "label": "安装 nano-pdf (uv)",
            },
          ],
      },
  }
---

# nano-pdf

使用 `nano-pdf`，通过自然语言指令对 PDF 中指定页面应用编辑。

## 快速开始

```bash
nano-pdf edit deck.pdf 1 "Change the title to 'Q3 Results' and fix the typo in the subtitle"
```

注意：

- 页码从 0 还是从 1 开始，取决于工具的版本/配置；如果结果差一页，换另一种方式重试。
- 发送前务必检查输出 PDF 是否正确。