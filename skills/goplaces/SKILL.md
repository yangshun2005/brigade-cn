---
name: goplaces
description: 通过 goplaces CLI 查询 Google Places API (New)，支持文本搜索、地点详情、解析和评论。适用于面向人的地点查询，或为脚本输出 JSON。
homepage: https://github.com/steipete/goplaces
metadata:
  {
    "brigade":
      {
        "emoji": "📍",
        "requires": { "bins": ["goplaces"], "env": ["GOOGLE_PLACES_API_KEY"] },
        "primaryEnv": "GOOGLE_PLACES_API_KEY",
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "steipete/tap/goplaces",
              "bins": ["goplaces"],
              "label": "安装 goplaces (brew)",
            },
          ],
      },
  }
---

# goplaces

现代化的 Google Places API (New) CLI。默认输出面向人的内容，脚本可用 `--json`。

安装

- Homebrew：`brew install steipete/tap/goplaces`

配置

- 必须设置 `GOOGLE_PLACES_API_KEY`。
- 可选：`GOOGLE_PLACES_BASE_URL`，用于测试或代理。

常用命令

- 搜索：`goplaces search "coffee" --open-now --min-rating 4 --limit 5`
- 位置偏好：`goplaces search "pizza" --lat 40.8 --lng -73.9 --radius-m 3000`
- 分页：`goplaces search "pizza" --page-token "NEXT_PAGE_TOKEN"`
- 解析：`goplaces resolve "Soho, London" --limit 5`
- 详情：`goplaces details <place_id> --reviews`
- JSON：`goplaces search "sushi" --json`

说明

- `--no-color` 或 `NO_COLOR` 可禁用 ANSI 颜色。
- 价格等级：0..4（免费 → 非常昂贵）。
- 类型过滤只发送第一个 `--type` 值（API 仅接受一个）。