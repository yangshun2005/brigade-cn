---
name: node-connect
description: 诊断 Android、iOS 和 macOS 配套应用的 Brigade 节点连接与配对故障。适用于二维码/设置码/手动连接失败、本地 Wi-Fi 可用但 VPS/tailnet 不可用，或错误信息提及 pairing required、unauthorized、bootstrap token invalid or expired、gateway.bind、gateway.remote.url、Tailscale 或 plugins.entries.device-pair.config.publicUrl 的情况。
---

# 节点连接

目标：找到从节点到网关的唯一真实路由，确认 Brigade 正在通告该路由，然后修复配对/认证。

## 先确定拓扑

提出修复方案前，先确定属于哪种情况：

- 同一机器 / 模拟器 / USB 隧道
- 同一 LAN / 本地 Wi-Fi
- 同一 Tailscale tailnet
- 公网 URL / 反向代理

不要混用。

- 本地 Wi-Fi 问题：除非确实需要远程访问，否则不要切换到 Tailscale。
- VPS / 远程网关问题：不要继续调试 `localhost` 或 LAN IP。

## 情况不明时先询问

如果配置不清楚或故障描述模糊，先简短提问再诊断。

询问：

- 预期使用哪条路由：同一机器、同一 LAN、Tailscale tailnet 还是公网 URL
- 使用的是二维码/设置码还是手动主机/端口
- 应用的确切文本/状态/错误，尽量原样引用
- `brigade devices list` 是否显示待处理的配对请求

不要根据 `can't connect` 猜测。

## 标准检查

优先使用 `brigade qr --json`。它使用的设置码载荷与 Android 扫描的相同。

```bash
brigade config get gateway.mode
brigade config get gateway.bind
brigade config get gateway.tailscale.mode
brigade config get gateway.remote.url
brigade config get gateway.auth.mode
brigade config get gateway.auth.allowTailscale
brigade config get plugins.entries.device-pair.config.publicUrl
brigade qr --json
brigade devices list
brigade nodes status
```

如果此 Brigade 实例指向远程网关，还需运行：

```bash
brigade qr --remote --json
```

如果涉及 Tailscale：

```bash
tailscale status --json
```

## 看结果，不要猜

`brigade qr --json` 成功意味着：

- `gatewayUrl`：这是应用应使用的实际端点。
- `urlSource`：告诉你哪条配置路径生效。

常见有效来源：

- `gateway.bind=lan`：仅同一 Wi-Fi / LAN
- `gateway.bind=tailnet`：直接 tailnet 访问
- `gateway.tailscale.mode=serve` 或 `gateway.tailscale.mode=funnel`：Tailscale 路由
- `plugins.entries.device-pair.config.publicUrl`：显式公网/反向代理路由
- `gateway.remote.url`：远程网关路由

## 根因对照

如果 `brigade qr --json` 提示 `Gateway is only bound to loopback`：

- 远程节点尚无法连接
- 先修复路由，再生成新的设置码
- 如果有效二维码路由仍是 loopback，`gateway.bind=auto` 不够
- 同一 LAN：使用 `gateway.bind=lan`
- 同一 tailnet：优先使用 `gateway.tailscale.mode=serve` 或 `gateway.bind=tailnet`
- 公网：设置真实的 `plugins.entries.device-pair.config.publicUrl` 或 `gateway.remote.url`

如果提示 `gateway.bind=tailnet set, but no tailnet IP was found`：

- 网关主机实际不在 Tailscale 上

如果提示 `qr --remote requires gateway.remote.url`：

- 远程模式配置不完整

如果应用提示 `pairing required`：

- 网络路由和认证正常
- 批准待处理设备

```bash
brigade devices list
brigade devices approve --latest
```

如果应用提示 `bootstrap token invalid or expired`：

- 设置码已过期
- 生成新的并重新扫描
- 任何 URL/认证修复后也要这样做

如果应用提示 `unauthorized`：

- 令牌/密码错误，或 Tailscale 预期不符
- 对于 Tailscale Serve，`gateway.auth.allowTailscale` 必须匹配预期流程
- 否则使用显式令牌/密码

## 快速判断

- 同一 Wi-Fi 配置 + 网关通告 `127.0.0.1`、`localhost` 或仅 loopback 配置：错误。
- 远程配置 + 设置/手动使用私有 LAN IP：错误。
- Tailnet 配置 + 网关通告 LAN IP 而非 MagicDNS / tailnet 路由：错误。
- 已设置公网 URL 但二维码仍通告其他内容：检查 `urlSource`；配置与你想的不符。
- `brigade devices list` 显示待处理请求：停止更改网络配置，先批准。

## 修复风格

回复一个明确诊断和一条路由。

如果信号不足，询问配置 + 应用确切文本，不要猜测。

好：

- `网关仍仅绑定 loopback，因此其他网络上的节点永远无法连接。启用 Tailscale Serve，重启网关，再次运行 brigade qr，重新扫描，然后批准待处理的设备配对。`

差：

- `可能是 LAN，可能是 Tailscale，可能是端口转发，可能是公网 URL。`