---
name: bluebubbles
description: 当你需要通过 BlueBubbles（推荐的 iMessage 集成）发送或管理 iMessage 时使用。调用通过通用 message 工具进行，channel="bluebubbles"。
metadata: { "brigade": { "emoji": "🫧", "requires": { "config": ["channels.bluebubbles"] } } }
---

# BlueBubbles 操作

## 概述

BlueBubbles 是 Brigade 推荐的 iMessage 集成。使用 `message` 工具并设置 `channel: "bluebubbles"` 来发送消息和管理 iMessage 对话：发送文本和附件、回应（tapback）、编辑/撤回、在话题中回复，以及管理群组成员/名称/图标。

## 需要收集的输入

- `target`（优先使用 `chat_guid:...`；也可使用 E.164 格式的 `+15551234567` 或 `user@example.com`）
- 发送/编辑/回复所需的 `message` 文本
- 回应/编辑/撤回/回复所需的 `messageId`
- 本地文件的附件 `path`，或 base64 的 `buffer` + `filename`

如果用户表述模糊（如"给我妈发条短信"），请询问收件人句柄或 chat guid，以及确切的消息内容。

## 操作

### 发送消息

```json
{
  "action": "send",
  "channel": "bluebubbles",
  "target": "+15551234567",
  "message": "hello from Brigade"
}
```

### 回应（tapback）

```json
{
  "action": "react",
  "channel": "bluebubbles",
  "target": "+15551234567",
  "messageId": "<message-guid>",
  "emoji": "❤️"
}
```

### 移除回应

```json
{
  "action": "react",
  "channel": "bluebubbles",
  "target": "+15551234567",
  "messageId": "<message-guid>",
  "emoji": "❤️",
  "remove": true
}
```

### 编辑已发送的消息

```json
{
  "action": "edit",
  "channel": "bluebubbles",
  "target": "+15551234567",
  "messageId": "<message-guid>",
  "message": "updated text"
}
```

### 撤回消息

```json
{
  "action": "unsend",
  "channel": "bluebubbles",
  "target": "+15551234567",
  "messageId": "<message-guid>"
}
```

### 回复特定消息

```json
{
  "action": "reply",
  "channel": "bluebubbles",
  "target": "+15551234567",
  "replyTo": "<message-guid>",
  "message": "replying to that"
}
```

### 发送附件

```json
{
  "action": "send",
  "channel": "bluebubbles",
  "target": "+15551234567",
  "path": "/tmp/photo.jpg",
  "message": "here you go"
}
```

使用 `message` 作为面向用户的文本。不要将 `caption` 当作 BlueBubbles/iMessage 的独立功能来依赖。

### 发送带 iMessage 特效的消息

```json
{
  "action": "sendWithEffect",
  "channel": "bluebubbles",
  "target": "+15551234567",
  "message": "big news",
  "effect": "balloons"
}
```

## 说明

- 需要网关配置 `channels.bluebubbles`（serverUrl/password/webhookPath）。
- 有 `chat_guid` 目标时优先使用（尤其是群聊）。
- 对于媒体 + 文本，以 `message` 为准。Brigade 可以接受 `caption` 作为别名，但 BlueBubbles/iMessage 没有可靠的附件说明原生概念。
- 实际上，带文本的媒体可能以附件加一条独立文本消息的形式到达，而非一个带说明的气泡。不要承诺精确的附件说明渲染效果。
- 如果必须使用频道专属的文件操作，规范操作名是 `upload-file`；`sendAttachment` 是旧版别名。
- BlueBubbles 支持丰富的操作，但部分操作依赖 macOS 版本（例如，编辑在 macOS 26 Tahoe 上可能失效）。
- 网关可能同时暴露短消息 ID 和完整消息 ID；完整 ID 在重启后更持久。
- 底层插件的开发者参考见 BlueBubbles 插件包的 README。

## 可以尝试的想法

- 用 tapback 回应以确认请求。
- 当用户引用特定消息时，在话题中回复。
- 发送文件附件并附带简短文本，预期 BlueBubbles 将该文本视为尽力而为的伴随文本，而非保证的原生说明。