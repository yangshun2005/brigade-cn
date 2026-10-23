---
name: slack
description: 当你需要通过 slack 工具从 Brigade 控制 Slack 时使用，包括对消息添加回应，或在 Slack 频道或私信中置顶/取消置顶条目。
metadata: { "brigade": { "emoji": "💬", "requires": { "config": ["channels.slack"] } } }
---

# Slack 操作

## 概述

使用 `slack` 添加回应、管理置顶、发送/编辑/删除消息，以及获取成员信息。该工具使用为 Brigade 配置的机器人令牌。

## 需要收集的输入

- `channelId` 和 `messageId`（Slack 消息时间戳，例如 `1712023032.1234`）。
- 添加回应时，需要 `emoji`（Unicode 或 `:name:`）。
- 发送消息时，需要 `to` 目标（`channel:<id>` 或 `user:<id>`）和 `content`。

消息上下文行包含 `slack message id` 和 `channel` 字段，可直接复用。

## 操作

### 操作组

| 操作组     | 默认   | 说明               |
| ---------- | ------ | ------------------ |
| reactions  | 启用   | 添加回应 + 列出回应 |
| messages   | 启用   | 读取/发送/编辑/删除 |
| pins       | 启用   | 置顶/取消置顶/列出  |
| memberInfo | 启用   | 成员信息           |
| emojiList  | 启用   | 自定义表情列表     |

### 对消息添加回应

```json
{
  "action": "react",
  "channelId": "C123",
  "messageId": "1712023032.1234",
  "emoji": "✅"
}
```

### 列出回应

```json
{
  "action": "reactions",
  "channelId": "C123",
  "messageId": "1712023032.1234"
}
```

### 发送消息

```json
{
  "action": "sendMessage",
  "to": "channel:C123",
  "content": "Hello from Brigade"
}
```

### 编辑消息

```json
{
  "action": "editMessage",
  "channelId": "C123",
  "messageId": "1712023032.1234",
  "content": "Updated text"
}
```

### 删除消息

```json
{
  "action": "deleteMessage",
  "channelId": "C123",
  "messageId": "1712023032.1234"
}
```

### 读取最近消息

```json
{
  "action": "readMessages",
  "channelId": "C123",
  "limit": 20
}
```

### 置顶消息

```json
{
  "action": "pinMessage",
  "channelId": "C123",
  "messageId": "1712023032.1234"
}
```

### 取消置顶消息

```json
{
  "action": "unpinMessage",
  "channelId": "C123",
  "messageId": "1712023032.1234"
}
```

### 列出置顶条目

```json
{
  "action": "listPins",
  "channelId": "C123"
}
```

### 成员信息

```json
{
  "action": "memberInfo",
  "userId": "U123"
}
```

### 表情列表

```json
{
  "action": "emojiList"
}
```

## 可尝试的用法

- 用 ✅ 回应来标记已完成的任务。
- 置顶关键决策或每周状态更新。