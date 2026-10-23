---
name: trello
description: 通过 Trello REST API 管理 Trello 看板、列表和卡片。
homepage: https://developer.atlassian.com/cloud/trello/rest/
metadata:
  {
    "brigade":
      {
        "emoji": "📋",
        "requires": { "bins": ["jq"], "env": ["TRELLO_API_KEY", "TRELLO_TOKEN"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "jq",
              "bins": ["jq"],
              "label": "安装 jq (brew)",
            },
          ],
      },
  }
---

# Trello 技能

直接在 Brigade 中管理 Trello 看板、列表和卡片。

## 设置

1. 获取 API key：https://trello.com/app-key
2. 生成 token（点击该页面上的 "Token" 链接）
3. 设置环境变量：
   ```bash
   export TRELLO_API_KEY="your-api-key"
   export TRELLO_TOKEN="your-token"
   ```

## 用法

所有命令均使用 curl 调用 Trello REST API。

### 列出看板

```bash
curl -s "https://api.trello.com/1/members/me/boards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" | jq '.[] | {name, id}'
```

### 列出看板中的列表

```bash
curl -s "https://api.trello.com/1/boards/{boardId}/lists?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" | jq '.[] | {name, id}'
```

### 列出列表中的卡片

```bash
curl -s "https://api.trello.com/1/lists/{listId}/cards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" | jq '.[] | {name, id, desc}'
```

### 创建卡片

```bash
curl -s -X POST "https://api.trello.com/1/cards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -d "idList={listId}" \
  -d "name=Card Title" \
  -d "desc=Card description"
```

### 将卡片移动到另一个列表

```bash
curl -s -X PUT "https://api.trello.com/1/cards/{cardId}?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -d "idList={newListId}"
```

### 为卡片添加评论

```bash
curl -s -X POST "https://api.trello.com/1/cards/{cardId}/actions/comments?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -d "text=Your comment here"
```

### 归档卡片

```bash
curl -s -X PUT "https://api.trello.com/1/cards/{cardId}?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -d "closed=true"
```

## 注意事项

- 看板/列表/卡片 ID 可在 Trello URL 中找到，或通过列表命令获取
- API key 和 token 可完全访问你的 Trello 账户——请妥善保管！
- 速率限制：每个 API key 每 10 秒 300 次请求；每个 token 每 10 秒 100 次请求；`/1/members` 端点限制为每 900 秒 100 次请求

## 示例

```bash
# 获取所有看板
curl -s "https://api.trello.com/1/members/me/boards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN&fields=name,id" | jq

# 按名称查找特定看板
curl -s "https://api.trello.com/1/members/me/boards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" | jq '.[] | select(.name | contains("Work"))'

# 获取看板上的所有卡片
curl -s "https://api.trello.com/1/boards/{boardId}/cards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" | jq '.[] | {name, list: .idList}'
```