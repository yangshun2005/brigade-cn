---
name: apple-notes
description: 在 macOS 上通过 `memo` CLI 管理 Apple Notes（创建、查看、编辑、删除、搜索、移动和导出笔记）。当用户要求 Brigade 添加笔记、列出笔记、搜索笔记或管理笔记文件夹时使用。
homepage: https://github.com/antoniorodr/memo
metadata:
  {
    "brigade":
      {
        "emoji": "📝",
        "os": ["darwin"],
        "requires": { "bins": ["memo"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "antoniorodr/memo/memo",
              "bins": ["memo"],
              "label": "通过 Homebrew 安装 memo",
            },
          ],
      },
  }
---

# Apple Notes CLI

使用 `memo notes` 直接从终端管理 Apple Notes。可创建、查看、编辑、删除、搜索笔记，在文件夹间移动笔记，以及导出为 HTML/Markdown。

安装配置

- 安装（Homebrew）：`brew tap antoniorodr/memo && brew install antoniorodr/memo/memo`
- 手动安装（pip）：`pip install .`（先克隆仓库）
- 仅限 macOS；如有提示，请授予 Notes.app 的自动化访问权限。

查看笔记

- 列出所有笔记：`memo notes`
- 按文件夹筛选：`memo notes -f "Folder Name"`
- 搜索笔记（模糊匹配）：`memo notes -s "query"`

创建笔记

- 添加新笔记：`memo notes -a`
  - 打开交互式编辑器撰写笔记。
- 带标题快速添加：`memo notes -a "Note Title"`

编辑笔记

- 编辑现有笔记：`memo notes -e`
  - 交互式选择要编辑的笔记。

删除笔记

- 删除笔记：`memo notes -d`
  - 交互式选择要删除的笔记。

移动笔记

- 将笔记移动到文件夹：`memo notes -m`
  - 交互式选择笔记和目标文件夹。

导出笔记

- 导出为 HTML/Markdown：`memo notes -ex`
  - 导出所选笔记；使用 Mistune 处理 markdown。

限制

- 无法编辑包含图片或附件的笔记。
- 交互式提示可能需要终端访问权限。

说明

- 仅限 macOS。
- 需要 Apple Notes.app 可访问。
- 如需自动化，请在系统设置 > 隐私与安全性 > 自动化中授予权限。