---
name: apple-reminders
description: 通过 remindctl CLI 管理 Apple Reminders（列出、添加、编辑、完成、删除）。支持列表、日期筛选和 JSON/纯文本输出。
homepage: https://github.com/steipete/remindctl
metadata:
  {
    "brigade":
      {
        "emoji": "⏰",
        "os": ["darwin"],
        "requires": { "bins": ["remindctl"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "steipete/tap/remindctl",
              "bins": ["remindctl"],
              "label": "通过 Homebrew 安装 remindctl",
            },
          ],
      },
  }
---

# Apple Reminders CLI (remindctl)

使用 `remindctl` 直接在终端管理 Apple Reminders。

## 何时使用

✅ **以下情况使用此技能：**

- 用户明确提到"提醒"或"提醒事项 App"
- 创建带截止日期、可同步到 iOS 的个人待办事项
- 管理 Apple Reminders 列表
- 用户希望任务出现在其 iPhone/iPad 的提醒事项 App 中

## 何时不使用

❌ **以下情况不要使用此技能：**

- 安排 Brigade 任务或告警 → 改用带 systemEvent 的 `cron` 工具
- 日历事件或预约 → 使用 Apple Calendar
- 项目/工作任务管理 → 使用 Notion、GitHub Issues 或任务队列
- 一次性通知 → 使用 `cron` 工具定时告警
- 用户说"提醒我"但实际指 Brigade 告警 → 先澄清

## 安装配置

- 安装：`brew install steipete/tap/remindctl`
- 仅限 macOS；提示时授予提醒事项权限
- 检查状态：`remindctl status`
- 请求授权：`remindctl authorize`

## 常用命令

### 查看提醒

```bash
remindctl                    # 今天的提醒
remindctl today              # 今天
remindctl tomorrow           # 明天
remindctl week               # 本周
remindctl overdue            # 已逾期
remindctl all                # 全部
remindctl 2026-01-04         # 指定日期
```

### 管理列表

```bash
remindctl list               # 列出所有列表
remindctl list Work          # 显示指定列表
remindctl list Projects --create    # 创建列表
remindctl list Work --delete        # 删除列表
```

### 创建提醒

```bash
remindctl add "Buy milk"
remindctl add --title "Call mom" --list Personal --due tomorrow
remindctl add --title "Meeting prep" --due "2026-02-15 09:00"
```

### 完成/删除

```bash
remindctl complete 1 2 3     # 按 ID 完成
remindctl delete 4A83 --force  # 按 ID 删除
```

### 输出格式

```bash
remindctl today --json       # JSON，用于脚本
remindctl today --plain      # TSV 格式
remindctl today --quiet      # 仅计数
```

## 日期格式

`--due` 和日期筛选接受以下格式：

- `today`、`tomorrow`、`yesterday`
- `YYYY-MM-DD`
- `YYYY-MM-DD HH:mm`
- ISO 8601（`2026-01-04T12:34:56Z`）

## 示例：澄清用户意图

用户："提醒我 2 小时后检查部署情况"

**询问：** "你想放到 Apple Reminders（同步到手机）还是作为 Brigade 告警（我在这里给你发消息）？"

- Apple Reminders → 使用此技能
- Brigade 告警 → 使用带 systemEvent 的 `cron` 工具