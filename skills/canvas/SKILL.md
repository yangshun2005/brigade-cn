---
name: canvas
description: 在已连接的、支持画布的显示界面上展示 HTML 内容——游戏、可视化、仪表盘、生成的 HTML、交互式演示。需要在屏幕上呈现可视化或交互式 Web 内容时使用。
---

# Canvas 技能

在已连接的 Brigade 节点（Mac 应用、iOS、Android）上展示 HTML 内容。

## 概述

canvas 工具可在任意已连接节点的画布视图中展示 Web 内容。适用于：

- 展示游戏、可视化、仪表盘
- 显示生成的 HTML 内容
- 交互式演示

## 工作原理

### 架构

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────┐
│  Canvas 主机    │────▶│   节点桥接       │────▶│  节点应用   │
│  (HTTP 服务器)  │     │  (TCP 服务器)    │     │ (Mac/iOS/   │
│  端口 18793     │     │  端口 18790      │     │  Android)   │
└─────────────────┘     └──────────────────┘     └─────────────┘
```

1. **Canvas 主机服务器**：从 `canvasHost.root` 目录提供静态 HTML/CSS/JS 文件
2. **节点桥接**：将画布 URL 传递给已连接的节点
3. **节点应用**：在 WebView 中渲染内容

### Tailscale 集成

Canvas 主机服务器根据 `gateway.bind` 设置绑定：

| 绑定模式   | 服务器绑定到        | 画布 URL 使用              |
| ---------- | ------------------- | -------------------------- |
| `loopback` | 127.0.0.1           | localhost（仅本地）        |
| `lan`      | LAN 接口            | LAN IP 地址                |
| `tailnet`  | Tailscale 接口      | Tailscale 主机名           |
| `auto`     | 最佳可用            | Tailscale > LAN > loopback |

**关键点：** `canvasHostHostForBridge` 由 `bridgeHost` 派生。绑定到 Tailscale 时，节点收到的 URL 如下：

```
http://<tailscale-hostname>:18793/__brigade__/canvas/<file>.html
```

这就是 localhost URL 不生效的原因——节点从桥接收到的是 Tailscale 主机名！

## 操作

| 操作       | 描述                         |
| ---------- | ---------------------------- |
| `present`  | 显示画布，可选目标 URL       |
| `hide`     | 隐藏画布                     |
| `navigate` | 导航到新 URL                 |
| `eval`     | 在画布中执行 JavaScript      |
| `snapshot` | 截取画布截图                 |

## 配置

在活动的 Brigade 配置文件（`$BRIGADE_CONFIG_PATH`，默认 `~/.brigade/brigade.json`）中：

```json
{
  "canvasHost": {
    "enabled": true,
    "port": 18793,
    "root": "/Users/you/brigade/canvas",
    "liveReload": true
  },
  "gateway": {
    "bind": "auto"
  }
}
```

### 实时重载

当 `liveReload: true`（默认）时，canvas 主机会：

- 监听根目录的变更（通过 chokidar）
- 向 HTML 文件注入 WebSocket 客户端
- 文件变更时自动重载已连接的画布

非常适合开发！

## 工作流程

### 1. 创建 HTML 内容

将文件放入 canvas 根目录（默认 `~/brigade/canvas/`）：

```bash
cat > ~/brigade/canvas/my-game.html << 'HTML'
<!DOCTYPE html>
<html>
<head><title>My Game</title></head>
<body>
  <h1>Hello Canvas!</h1>
</body>
</html>
HTML
```

### 2. 查找 canvas 主机 URL

检查网关的绑定方式：

```bash
CONFIG_PATH="${BRIGADE_CONFIG_PATH:-${BRIGADE_STATE_DIR:-$HOME/.brigade}/brigade.json}"
cat "$CONFIG_PATH" | jq '.gateway.bind'
```

然后构造 URL：

- **loopback**：`http://127.0.0.1:18793/__brigade__/canvas/<file>.html`
- **lan/tailnet/auto**：`http://<hostname>:18793/__brigade__/canvas/<file>.html`

查找 Tailscale 主机名：

```bash
tailscale status --json | jq -r '.Self.DNSName' | sed 's/\.$//'
```

### 3. 查找已连接的节点

```bash
brigade nodes list
```

查找具备画布能力的 Mac/iOS/Android 节点。

### 4. 展示内容

```
canvas action:present node:<node-id> target:<full-url>
```

**示例：**

```
canvas action:present node:mac-63599bc4-b54d-4392-9048-b97abd58343a target:http://peters-mac-studio-1.sheep-coho.ts.net:18793/__brigade__/canvas/snake.html
```

### 5. 导航、截图或隐藏

```
canvas action:navigate node:<node-id> url:<new-url>
canvas action:snapshot node:<node-id>
canvas action:hide node:<node-id>
```

## 调试

### 白屏 / 内容未加载

**原因：** 服务器绑定与节点预期之间的 URL 不匹配。

**调试步骤：**

1. 检查服务器绑定：`CONFIG_PATH="${BRIGADE_CONFIG_PATH:-${BRIGADE_STATE_DIR:-$HOME/.brigade}/brigade.json}"; cat "$CONFIG_PATH" | jq '.gateway.bind'`
2. 检查 canvas 所在端口：`lsof -i :18793`
3. 直接测试 URL：`curl http://<hostname>:18793/__brigade__/canvas/<file>.html`

**解决方案：** 使用与绑定模式匹配的完整主机名，而非 localhost。

### "node required" 错误

始终指定 `node:<node-id>` 参数。

### "node not connected" 错误

节点离线。使用 `brigade nodes list` 查找在线节点。

### 内容未更新

如果实时重载不工作：

1. 检查配置中 `liveReload: true`
2. 确保文件位于 canvas 根目录
3. 检查日志中的监听器错误

## URL 路径结构

Canvas 主机从 `/__brigade__/canvas/` 前缀提供内容：

```
http://<host>:18793/__brigade__/canvas/index.html  → ~/brigade/canvas/index.html
http://<host>:18793/__brigade__/canvas/games/snake.html → ~/brigade/canvas/games/snake.html
```

`/__brigade__/canvas/` 前缀由 `CANVAS_HOST_PATH` 常量定义。

## 提示

- 保持 HTML 自包含（内联 CSS/JS）以获得最佳效果
- 使用默认的 index.html 作为测试页（含桥接诊断）
- 画布会一直保留，直到你 `hide` 它或导航离开
- 实时重载让开发更高效——保存即更新！
- A2UI JSON 推送尚在开发中——目前请使用 HTML 文件