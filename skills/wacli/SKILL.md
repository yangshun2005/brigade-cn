---
name: wacli
description: 通过 wacli CLI 向他人发送 WhatsApp 消息，或搜索/同步 WhatsApp 历史记录（不用于普通用户聊天）。
homepage: https://wacli.sh
metadata:
  {
    "brigade":
      {
        "emoji": "📱",
        "requires": { "bins": ["wacli"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "steipete/tap/wacli",
              "bins": ["wacli"],
              "label": "安装 wacli (brew)",
            },
            {
              "id": "go",
              "kind": "go",
              "module": "github.com/steipete/wacli/cmd/wacli@latest",
              "bins": ["wacli"],
              "label": "安装 wacli (go)",
            },
          ],
      },
  }
---

# wacli

仅在用户明确要求你向他人发送 WhatsApp 消息，或要求同步/搜索 WhatsApp 历史记录时使用 `wacli`。
不要将 `wacli` 用于普通用户聊天；Brigade 会自动路由 WhatsApp 对话。
如果用户正在 WhatsApp 上与你聊天，除非他们要求你联系第三方，否则不应使用此工具。

安全

- 需要明确收件人 + 消息文本。
- 发送前确认收件人 + 消息。
- 如有任何歧义，先提出澄清问题。

认证 + 同步

- `wacli auth`（扫码登录 + 初始同步）
- `wacli sync --follow`（持续同步）
- `wacli doctor`

查找聊天 + 消息

- `wacli chats list --limit 20 --query "name or number"`
- `wacli messages search "query" --limit 20 --chat <jid>`
- `wacli messages search "invoice" --after 2025-01-01 --before 2025-12-31`

历史回填

- `wacli history backfill --chat <jid> --requests 2 --count 50`

发送

- 文本：`wacli send text --to "+14155551212" --message "Hello! Are you free at 3pm?"`
- 群组：`wacli send text --to "1234567890-123456789@g.us" --message "Running 5 min late."`
- 文件：`wacli send file --to "+14155551212" --file /path/agenda.pdf --caption "Agenda"`

说明

- 存储目录：`~/.wacli`（可用 `--store` 覆盖）。
- 解析时使用 `--json` 获取机器可读输出。
- 回填需要手机在线；结果为尽力而为。
- 日常用户聊天不需要 WhatsApp CLI；它用于向他人发送消息。
- JID：直接聊天形如 `<number>@s.whatsapp.net`；群组形如 `<id>@g.us`（用 `wacli chats list` 查找）。