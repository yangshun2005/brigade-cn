---
name: xurl
description: 用于向 X（Twitter）API 发起认证请求的 CLI 工具。当你需要发帖、回复、引用、搜索、读取帖子、管理关注者、发送私信、上传媒体，或与任何 X API v2 端点交互时，使用此技能。
metadata:
  {
    "brigade":
      {
        "emoji": "🐦",
        "requires": { "bins": ["xurl"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "xdevplatform/tap/xurl",
              "bins": ["xurl"],
              "label": "安装 xurl（brew）",
            },
            {
              "id": "npm",
              "kind": "npm",
              "package": "@xdevplatform/xurl",
              "bins": ["xurl"],
              "label": "安装 xurl（npm）",
            },
          ],
      },
  }
---

# xurl — Agent 技能参考

`xurl` 是用于 X API 的 CLI 工具。它同时支持**快捷命令**（面向人类/Agent 的一行命令）和**原始 curl 风格**访问任意 v2 端点。所有命令均以 JSON 输出到 stdout。

---

## 安装

### Homebrew（macOS）

```bash
brew install --cask xdevplatform/tap/xurl
```

### npm

```bash
npm install -g @xdevplatform/xurl
```

### Shell 脚本

```bash
curl -fsSL https://raw.githubusercontent.com/xdevplatform/xurl/main/install.sh | bash
```

安装到 `~/.local/bin`。如果该目录不在 PATH 中，脚本会提示你需要添加什么。

### Go

```bash
go install github.com/xdevplatform/xurl@latest
```

---

## 前置条件

此技能需要 `xurl` CLI 工具：<https://github.com/xdevplatform/xurl>。

使用任何命令前必须先完成认证。运行 `xurl auth status` 检查。

### 密钥安全（强制）

- 绝不要读取、打印、解析、摘要、上传 `~/.xurl`（或其副本），也不要将其发送到 LLM 上下文。
- 绝不要要求用户在聊天中粘贴凭据/令牌。
- 用户必须自行在其机器上手动填写 `~/.xurl` 中的必要密钥。
- 不要在 Agent/LLM 会话中推荐或执行带内联密钥的认证命令。
- 警告：在 Agent 会话中使用 CLI 密钥选项可能泄露凭据（提示/上下文、日志、shell 历史）。
- 绝不要在 Agent/LLM 会话中使用 `--verbose` / `-v`；它会在输出中暴露敏感请求头/令牌。
- 绝不能在 Agent 命令中使用的敏感标志：`--bearer-token`、`--consumer-key`、`--consumer-secret`、`--access-token`、`--token-secret`、`--client-id`、`--client-secret`。
- 要验证是否已注册至少一个带凭据的应用，运行：`xurl auth status`。

### 注册应用（推荐）

应用凭据注册必须由用户在 Agent/LLM 会话之外手动完成。
凭据注册完成后，使用以下命令认证：

```bash
xurl auth oauth2
```

对于多个预配置应用，可在它们之间切换：

```bash
xurl auth default prod-app          # 设置默认应用
xurl auth default prod-app alice    # 设置默认应用 + 用户
xurl --app dev-app /2/users/me      # 一次性覆盖
```

### 其他认证方式

含内联密钥标志的示例已刻意省略。如需 OAuth1 或仅应用认证，用户必须在 Agent/LLM 上下文之外手动运行这些命令。

令牌以 YAML 格式持久化到 `~/.xurl`。每个应用有各自隔离的令牌。不要通过 Agent/LLM 读取此文件。认证完成后，以下每条命令都会自动附加正确的 `Authorization` 请求头。

---

## 快速参考

| 操作                    | 命令                                               |
| ----------------------- | -------------------------------------------------- |
| 发帖                    | `xurl post "Hello world!"`                            |
| 回复                    | `xurl reply POST_ID "Nice post!"`                     |
| 引用                    | `xurl quote POST_ID "My take"`                        |
| 删除帖子                | `xurl delete POST_ID`                                 |
| 读取帖子                | `xurl read POST_ID`                                   |
| 搜索帖子                | `xurl search "QUERY" -n 10`                           |
| 我是谁                  | `xurl whoami`                                         |
| 查找用户                | `xurl user @handle`                                   |
| 主页时间线              | `xurl timeline -n 20`                                 |
| 提及                    | `xurl mentions -n 10`                                 |
| 点赞                    | `xurl like POST_ID`                                   |
| 取消点赞                | `xurl unlike POST_ID`                                 |
| 转发                    | `xurl repost POST_ID`                                 |
| 撤销转发                | `xurl unrepost POST_ID`                               |
| 收藏                    | `xurl bookmark POST_ID`                               |
| 移除收藏                | `xurl unbookmark POST_ID`                             |
| 列出收藏                | `xurl bookmarks -n 10`                                |
| 列出点赞                | `xurl likes -n 10`                                    |
| 关注                    | `xurl follow @handle`                                 |
| 取消关注                | `xurl unfollow @handle`                               |
| 列出关注中              | `xurl following -n 20`                                |
| 列出关注者              | `xurl followers -n 20`                                |
| 屏蔽                    | `xurl block @handle`                                  |
| 取消屏蔽                | `xurl unblock @handle`                                |
| 静音                    | `xurl mute @handle`                                   |
| 取消静音                | `xurl unmute @handle`                                 |
| 发送私信                | `xurl dm @handle "message"`                           |
| 列出私信                | `xurl dms -n 10`                                      |
| 上传媒体                | `xurl media upload path/to/file.mp4`                  |
| 媒体状态                | `xurl media status MEDIA_ID`                          |
| **应用管理**            |                                                       |
| 注册应用                | 手动，在 Agent 之外（不要通过 Agent 传递密钥） |
| 列出应用                | `xurl auth apps list`                                 |
| 更新应用凭据            | 手动，在 Agent 之外（不要通过 Agent 传递密钥） |
| 移除应用                | `xurl auth apps remove NAME`                          |
| 设置默认（交互式）      | `xurl auth default`                                   |
| 设置默认（命令式）      | `xurl auth default APP_NAME [USERNAME]`               |
| 按请求使用应用          | `xurl --app NAME /2/users/me`                         |
| 认证状态                | `xurl auth status`                                    |

> **帖子 ID 与 URL：** 上文任何出现 `POST_ID` 的地方，你也可以粘贴完整的帖子 URL（例如 `https://x.com/user/status/1234567890`）——xurl 会自动提取 ID。

> **用户名：** 开头的 `@` 可选。`@elonmusk` 和 `elonmusk` 均可。

---

## 命令详情

### 发帖

```bash
# 简单发帖
xurl post "Hello world!"

# 带媒体发帖（先上传，再附加）
xurl media upload photo.jpg          # → 记下响应中的 media_id
xurl post "Check this out" --media-id MEDIA_ID

# 多个媒体
xurl post "Thread pics" --media-id 111 --media-id 222

# 回复帖子（通过 ID 或 URL）
xurl reply 1234567890 "Great point!"
xurl reply https://x.com/user/status/1234567890 "Agreed!"

# 带媒体回复
xurl reply 1234567890 "Look at this" --media-id MEDIA_ID

# 引用帖子
xurl quote 1234567890 "Adding my thoughts"

# 删除自己的帖子
xurl delete 1234567890
```

### 读取

```bash
# 读取单条帖子（返回作者、文本、指标、实体）
xurl read 1234567890
xurl read https://x.com/user/status/1234567890

# 搜索近期帖子（默认 10 条结果）
xurl search "golang"
xurl search "from:elonmusk" -n 20
xurl search "#buildinpublic lang:en" -n 15
```

### 用户信息

```bash
# 自己的个人资料
xurl whoami

# 查找任意用户
xurl user elonmusk
xurl user @XDevelopers
```

### 时间线与提及

```bash
# 主页时间线（倒序）
xurl timeline
xurl timeline -n 25

# 你的提及
xurl mentions
xurl mentions -n 20
```

### 互动

```bash
# 点赞 / 取消点赞
xurl like 1234567890
xurl unlike 1234567890

# 转发 / 撤销
xurl repost 1234567890
xurl unrepost 1234567890

# 收藏 / 移除
xurl bookmark 1234567890
xurl unbookmark 1234567890

# 列出你的收藏 / 点赞
xurl bookmarks -n 20
xurl likes -n 20
```

### 社交关系

```bash
# 关注 / 取消关注
xurl follow @XDevelopers
xurl unfollow @XDevelopers

# 列出你关注的人 / 你的关注者
xurl following -n 50
xurl followers -n 50

# 列出其他用户的关注/关注者
xurl following --of elonmusk -n 20
xurl followers --of elonmusk -n 20

# 屏蔽 / 取消屏蔽
xurl block @spammer
xurl unblock @spammer

# 静音 / 取消静音
xurl mute @annoying
xurl unmute @annoying
```

### 私信

```bash
# 发送私信
xurl dm @someuser "Hey, saw your post!"

# 列出近期私信事件
xurl dms
xurl dms -n 25
```

### 媒体上传

```bash
# 上传文件（自动检测图片/视频类型）
xurl media upload photo.jpg
xurl media upload video.mp4

# 显式指定类型和类别
xurl media upload --media-type image/jpeg --category tweet_image photo.jpg

# 检查处理状态（视频需要服务端处理）
xurl media status MEDIA_ID
xurl media status --wait MEDIA_ID    # 轮询直到完成

# 完整流程：上传后发帖
xurl media upload meme.png           # 响应包含 media id
xurl post "lol" --media-id MEDIA_ID
```

---

## 全局标志

这些标志适用于所有命令：

| 标志         | 短写 | 描述                                                        |
| ------------ | ----- | ------------------------------------------------------------------ |
| `--app`      |       | 为此请求使用指定的已注册应用（覆盖默认值） |
| `--auth`     |       | 强制认证类型：`oauth1`、`oauth2` 或 `app`                      |
| `--username` | `-u`  | 使用哪个 OAuth2 账户（如果你有多个）                 |
| `--verbose`  | `-v`  | 在 Agent/LLM 会话中禁用（会泄露认证请求头/令牌）     |
| `--trace`    | `-t`  | 添加 `X-B3-Flags: 1` 追踪请求头                                   |

---

## 原始 API 访问

快捷命令覆盖了最常见的操作。其他任何操作，使用 xurl 的原始 curl 风格模式——它适用于**任何** X API v2 端点：

```bash
# GET 请求（默认）
xurl /2/users/me

# 带 JSON 体的 POST
xurl -X POST /2/tweets -d '{"text":"Hello world!"}'

# PUT、PATCH、DELETE
xurl -X DELETE /2/tweets/1234567890

# 自定义请求头
xurl -H "Content-Type: application/json" /2/some/endpoint

# 强制流式模式
xurl -s /2/tweets/search/stream

# 完整 URL 也可用
xurl https://api.x.com/2/users/me
```

---

## 流式传输

流式端点会自动检测。已知的流式端点包括：

- `/2/tweets/search/stream`
- `/2/tweets/sample/stream`
- `/2/tweets/sample10/stream`

你可以用 `-s` 强制对任意端点启用流式：

```bash
xurl -s /2/some/endpoint
```

---

## 输出格式

所有命令均以 **JSON** 输出到 stdout，带语法高亮的格式化打印。输出结构与 X API v2 响应格式一致。典型响应如下：

```json
{
  "data": {
    "id": "1234567890",
    "text": "Hello world!"
  }
}
```

错误也以 JSON 返回：

```json
{
  "errors": [
    {
      "message": "Not authorized",
      "code": 403
    }
  ]
}
```

---

## 常见工作流

### 带图片发帖

```bash
# 1. 上传图片
xurl media upload photo.jpg
# 2. 从响应中复制 media_id，然后发帖
xurl post "Check out this photo!" --media-id MEDIA_ID
```

### 回复对话

```bash
# 1. 读取帖子以了解上下文
xurl read https://x.com/user/status/1234567890
# 2. 回复
xurl reply 1234567890 "Here are my thoughts..."
```

### 搜索并互动

```bash
# 1. 搜索相关帖子
xurl search "topic of interest" -n 10
# 2. 点赞一条有趣的
xurl like POST_ID_FROM_RESULTS
# 3. 回复它
xurl reply POST_ID_FROM_RESULTS "Great point!"
```

### 查看你的活动

```bash
# 查看你是谁
xurl whoami
# 检查你的提及
xurl mentions -n 20
# 检查你的时间线
xurl timeline -n 20
```

### 设置多个应用

```bash
# 应用凭据必须已在 Agent/LLM 上下文之外手动配置。
# 在每个预配置应用上认证用户
xurl auth default prod
xurl auth oauth2                       # 在 prod 应用上认证

xurl auth default staging
xurl auth oauth2                       # 在 staging 应用上认证

# 在它们之间切换
xurl auth default prod alice           # prod 应用，alice 用户
xurl --app staging /2/users/me         # 对 staging 的一次性请求
```

---

## 错误处理

- 任何错误均返回非零退出码。
- API 错误以 JSON 打印到 stdout（因此你仍可解析它们）。
- 认证错误会建议重新运行 `xurl auth oauth2` 或检查你的令牌。
- 如果命令需要你的用户 ID（点赞、转发、收藏、关注等），xurl 会自动通过 `/2/users/me` 获取。如果失败，你会看到认证错误。

---

## 注意事项

- **速率限制：** X API 按端点实施速率限制。如果收到 429 错误，请等待后重试。写端点（发帖、回复、点赞、转发）的限制比读端点更严格。
- **作用域：** OAuth 2.0 令牌以宽泛作用域请求。如果某个操作收到 403，你的令牌可能缺少所需作用域——重新运行 `xurl auth oauth2` 获取新令牌。
- **令牌刷新：** OAuth 2.0 令牌过期后自动刷新。无需手动干预。
- **多应用：** 每个应用有各自隔离的凭据和令牌。在 Agent/LLM 上下文之外手动配置凭据，然后用 `xurl auth default` 或 `--app` 切换。
- **多账户：** 每个应用可认证多个 OAuth 2.0 账户，用 `--username` / `-u` 在它们之间切换，或用 `xurl auth default APP USER` 设置默认。
- **默认用户：** 未给出 `-u` 标志时，xurl 使用当前应用的默认用户（通过 `xurl auth default` 设置）。如果未设置默认用户，则使用第一个可用令牌。
- **令牌存储：** `~/.xurl` 为 YAML 格式。每个应用存储各自的凭据和令牌。绝不要读取此文件或将其发送到 LLM 上下文。