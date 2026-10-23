---
name: discord
description: "通过 message 工具执行 Discord 操作（channel=discord）。"
metadata: { "brigade": { "emoji": "🎮", "requires": { "config": ["channels.discord.token"] } } }
allowed-tools: ["message"]
---

# Discord（通过 `message`）

使用 `message` 工具。不向 agent 暴露 Discord 专用的 `discord` 工具。

## 必须遵守

- 始终设置：`channel: "discord"`。
- 遵守开关限制：`channels.discord.actions.*`（部分默认关闭：`roles`、`moderation`、`presence`、`channels`）。
- 优先使用显式 ID：`guildId`、`channelId`、`messageId`、`userId`。
- 多账号：可选 `accountId`。

## 指南

- 外发 Discord 消息中避免使用 Markdown 表格。
- 提及用户使用 `<@USER_ID>`。
- 富 UI 优先使用 Discord 组件 v2（`components`）；仅在必要时使用旧版 `embeds`。

## 目标

- 发送类操作：`to: "channel:<id>"` 或 `to: "user:<id>"`。
- 消息级操作：`channelId: "<id>"`（或 `to`）+ `messageId: "<id>"`。

## 常用操作（示例）

发送消息：

```json
{
  "action": "send",
  "channel": "discord",
  "to": "channel:123",
  "message": "hello",
  "silent": true
}
```

发送带媒体：

```json
{
  "action": "send",
  "channel": "discord",
  "to": "channel:123",
  "message": "see attachment",
  "media": "file:///tmp/example.png"
}
```

- 可选 `silent: true` 以抑制 Discord 通知。

使用组件 v2 发送（富 UI 推荐）：

```json
{
  "action": "send",
  "channel": "discord",
  "to": "channel:123",
  "message": "Status update",
  "components": "[Carbon v2 components]"
}
```

- `components` 需要来自 JS/TS 集成的 Carbon 组件实例（Container、TextDisplay 等）。
- 不要将 `components` 与 `embeds` 混用（Discord 拒绝 v2 + embeds）。

旧版 embeds（不推荐）：

```json
{
  "action": "send",
  "channel": "discord",
  "to": "channel:123",
  "message": "Status update",
  "embeds": [{ "title": "Legacy", "description": "Embeds are legacy." }]
}
```

- 存在组件 v2 时，`embeds` 会被忽略。

添加反应：

```json
{
  "action": "react",
  "channel": "discord",
  "channelId": "123",
  "messageId": "456",
  "emoji": "✅"
}
```

读取：

```json
{
  "action": "read",
  "channel": "discord",
  "to": "channel:123",
  "limit": 20
}
```

编辑 / 删除：

```json
{
  "action": "edit",
  "channel": "discord",
  "channelId": "123",
  "messageId": "456",
  "message": "fixed typo"
}
```

```json
{
  "action": "delete",
  "channel": "discord",
  "channelId": "123",
  "messageId": "456"
}
```

投票：

```json
{
  "action": "poll",
  "channel": "discord",
  "to": "channel:123",
  "pollQuestion": "Lunch?",
  "pollOption": ["Pizza", "Sushi", "Salad"],
  "pollMulti": false,
  "pollDurationHours": 24
}
```

置顶：

```json
{
  "action": "pin",
  "channel": "discord",
  "channelId": "123",
  "messageId": "456"
}
```

子区：

```json
{
  "action": "thread-create",
  "channel": "discord",
  "channelId": "123",
  "messageId": "456",
  "threadName": "bug triage"
}
```

搜索：

```json
{
  "action": "search",
  "channel": "discord",
  "guildId": "999",
  "query": "release notes",
  "channelIds": ["123", "456"],
  "limit": 10
}
```

在线状态（通常受限）：

```json
{
  "action": "set-presence",
  "channel": "discord",
  "activityType": "playing",
  "activityName": "with fire",
  "status": "online"
}
```

## 写作风格（Discord）

- 简短、口语化、少客套。
- 不使用 Markdown 表格。
- 提及用户使用 `<@USER_ID>`。