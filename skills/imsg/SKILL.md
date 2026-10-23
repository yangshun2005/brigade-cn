---
name: imsg
description: 用于列出聊天、查看历史记录和通过 Messages.app 发送消息的 iMessage/SMS CLI。
homepage: https://imsg.to
metadata:
  {
    "brigade":
      {
        "emoji": "📨",
        "os": ["darwin"],
        "requires": { "bins": ["imsg"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "steipete/tap/imsg",
              "bins": ["imsg"],
              "label": "安装 imsg (brew)",
            },
          ],
      },
  }
---

# imsg

使用 `imsg` 通过 macOS Messages.app 读取和发送 iMessage/SMS。

## 何时使用

✅ **以下情况使用此技能：**

- 用户明确要求发送 iMessage 或 SMS
- 读取 iMessage 对话历史
- 查看最近的 Messages.app 聊天
- 向电话号码或 Apple ID 发送消息

## 何时不使用

❌ **以下情况不要使用此技能：**

- Telegram 消息 → 使用 `message` 工具并指定 `channel:telegram`
- Signal 消息 → 如已配置，使用 Signal 渠道
- WhatsApp 消息 → 如已配置，使用 WhatsApp 渠道
- Discord 消息 → 使用 `message` 工具并指定 `channel:discord`
- Slack 消息 → 使用 `slack` 技能
- 群聊管理（添加/移除成员）→ 不支持
- 批量/群发消息 → 必须先与用户确认
- 在当前对话中回复 → 正常回复即可（Brigade 自动路由）

## 要求

- macOS 且 Messages.app 已登录
- 终端需要完全磁盘访问权限
- Messages.app 需要自动化权限（用于发送）

## 常用命令

### 列出聊天

```bash
imsg chats --limit 10 --json
```

### 查看历史记录

```bash
# 按聊天 ID
imsg history --chat-id 1 --limit 20 --json

# 包含附件信息
imsg history --chat-id 1 --limit 20 --attachments --json
```

### 监听新消息

```bash
imsg watch --chat-id 1 --attachments
```

### 发送消息

```bash
# 仅文本
imsg send --to "+14155551212" --text "Hello!"

# 带附件
imsg send --to "+14155551212" --text "Check this out" --file /path/to/image.jpg

# 指定服务
imsg send --to "+14155551212" --text "Hi" --service imessage
imsg send --to "+14155551212" --text "Hi" --service sms
```

## 服务选项

- `--service imessage` — 强制使用 iMessage（要求对方已开通 iMessage）
- `--service sms` — 强制使用 SMS（绿色气泡）
- `--service auto` — 由 Messages.app 决定（默认）

## 安全规则

1. **发送前务必确认收件人和消息内容**
2. **未经用户明确批准，绝不向未知号码发送消息**
3. **谨慎处理附件** — 确认文件路径存在
4. **自我限速** — 不要滥发

## 示例工作流

用户："发短信告诉妈妈我会迟到"

```bash
# 1. 查找妈妈的聊天
imsg chats --limit 20 --json | jq '.[] | select(.displayName | contains("Mom"))'

# 2. 与用户确认
# "找到妈妈 +1555123456。通过 iMessage 发送'我会迟到'？"

# 3. 确认后发送
imsg send --to "+1555123456" --text "I'll be late"
```