---
name: obsidian
description: 操作 Obsidian 仓库（纯 Markdown 笔记），并通过 obsidian-cli 自动化。
homepage: https://help.obsidian.md
metadata:
  {
    "brigade":
      {
        "emoji": "💎",
        "requires": { "bins": ["obsidian-cli"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "yakitrak/yakitrak/obsidian-cli",
              "bins": ["obsidian-cli"],
              "label": "安装 obsidian-cli（brew）",
            },
          ],
      },
  }
---

# Obsidian

Obsidian 仓库 = 磁盘上的普通文件夹。

仓库结构（典型）

- 笔记：`*.md`（纯文本 Markdown；可用任意编辑器编辑）
- 配置：`.obsidian/`（工作区 + 插件设置；通常不要用脚本改动）
- 画布：`*.canvas`（JSON）
- 附件：你在 Obsidian 设置中选择的文件夹（图片/PDF 等）

## 查找当前活跃的仓库

Obsidian 桌面版在此记录仓库（权威来源）：

- `~/Library/Application Support/obsidian/obsidian.json`

`obsidian-cli` 从该文件解析仓库；仓库名通常是**文件夹名**（路径后缀）。

快速确认“哪个仓库活跃 / 笔记在哪？”

- 若已设置默认：`obsidian-cli print-default --path-only`
- 否则读取 `~/Library/Application Support/obsidian/obsidian.json`，使用 `"open": true` 的仓库条目。

注意

- 多仓库很常见（iCloud 与 `~/Documents`、工作/个人等）。不要猜，读配置。
- 避免在脚本中硬编码仓库路径；优先读配置或使用 `print-default`。

## obsidian-cli 快速上手

设置默认仓库（一次）：

- `obsidian-cli set-default "<vault-folder-name>"`
- `obsidian-cli print-default` / `obsidian-cli print-default --path-only`

搜索

- `obsidian-cli search "query"`（笔记名）
- `obsidian-cli search-content "query"`（笔记内容；显示片段 + 行号）

创建

- `obsidian-cli create "Folder/New note" --content "..." --open`
- 需要 Obsidian URI 处理器（`obsidian://…`）可用（已安装 Obsidian）。
- 避免通过 URI 在“隐藏”点文件夹（如 `.something/...`）下创建笔记；Obsidian 可能拒绝。

移动/重命名（安全重构）

- `obsidian-cli move "old/path/note" "new/path/note"`
- 更新整个仓库中的 `[[wikilinks]]` 和常见 Markdown 链接（这是相比 `mv` 的主要优势）。

删除

- `obsidian-cli delete "path/note"`

合适时优先直接编辑：打开 `.md` 文件修改；Obsidian 会自动识别。