---
name: bear-notes
description: 通过 grizzly CLI 创建、搜索和管理 Bear 笔记。
homepage: https://bear.app
metadata:
  {
    "brigade":
      {
        "emoji": "🐻",
        "os": ["darwin"],
        "requires": { "bins": ["grizzly"] },
        "install":
          [
            {
              "id": "go",
              "kind": "go",
              "module": "github.com/tylerwince/grizzly/cmd/grizzly@latest",
              "bins": ["grizzly"],
              "label": "安装 grizzly（go）",
            },
          ],
      },
  }
---

# Bear Notes

使用 `grizzly` 在 macOS 上创建、读取和管理 Bear 笔记。

要求

- 已安装并运行 Bear 应用
- 部分操作（add-text、tags、open-note --selected）需要 Bear 应用令牌（存储在 `~/.config/grizzly/token`）

## 获取 Bear 令牌

需要令牌的操作（add-text、tags、open-note --selected）需提供认证令牌：

1. 打开 Bear → Help → API Token → Copy Token
2. 保存令牌：`echo "YOUR_TOKEN" > ~/.config/grizzly/token`

## 常用命令

创建笔记

```bash
echo "Note content here" | grizzly create --title "My Note" --tag work
grizzly create --title "Quick Note" --tag inbox < /dev/null
```

按 ID 打开/读取笔记

```bash
grizzly open-note --id "NOTE_ID" --enable-callback --json
```

向笔记追加文本

```bash
echo "Additional content" | grizzly add-text --id "NOTE_ID" --mode append --token-file ~/.config/grizzly/token
```

列出所有标签

```bash
grizzly tags --enable-callback --json --token-file ~/.config/grizzly/token
```

搜索笔记（通过 open-tag）

```bash
grizzly open-tag --name "work" --enable-callback --json
```

## 选项

常用标志：

- `--dry-run` — 预览 URL，不执行
- `--print-url` — 显示 x-callback-url
- `--enable-callback` — 等待 Bear 响应（读取数据时必需）
- `--json` — 以 JSON 输出（使用回调时）
- `--token-file PATH` — Bear API 令牌文件路径

## 配置

Grizzly 按以下优先级读取配置：

1. CLI 标志
2. 环境变量（`GRIZZLY_TOKEN_FILE`、`GRIZZLY_CALLBACK_URL`、`GRIZZLY_TIMEOUT`）
3. 当前目录下的 `.grizzly.toml`
4. `~/.config/grizzly/config.toml`

`~/.config/grizzly/config.toml` 示例：

```toml
token_file = "~/.config/grizzly/token"
callback_url = "http://127.0.0.1:42123/success"
timeout = "5s"
```

## 说明

- Bear 必须处于运行状态，命令才能生效
- 笔记 ID 是 Bear 的内部标识符（可在笔记信息中或通过回调查看）
- 需要从 Bear 读回数据时，使用 `--enable-callback`
- 部分操作需要有效令牌（add-text、tags、open-note --selected）