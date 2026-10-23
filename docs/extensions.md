# 编写 Brigade 扩展

**扩展**是 Brigade 可扩展性的单元。一个扩展（一个*模块*）可以注册任意组合的智能体级能力（工具、Hook、斜杠命令、模型提供方）和产品级能力（频道、语音、媒体、记忆后端、后台服务、HTTP 路由、网关 RPC 方法）——全部通过单个 `register(b)` 函数完成。

本指南介绍如何编写扩展、Brigade 如何发现它、安全模型，以及帮助你构建和调试它的 CLI 命令。

---

## 模块的形态

每个扩展导出一个用 `defineModule` 构建的模块：

```ts
import { defineModule } from "brigade/extension-sdk";

export default defineModule({
  id: "my-extension",
  register(b) {
    // register capabilities through b.* here
  },
});
```

- `id` 是稳定的 kebab-case 标识符（也是文件夹名）。
- `register(b)` 接收扩展上下文 `b`。你注册的一切都会被**记录**——智能体级注册会在每个智能体回合中重放，产品级注册由网关在启动时启动一次。

你也可以导出一个模块数组，或附加可选的 `manifest`、`requiresEnv`、`configSchema` 和 `eligible()` 门控。完整接口见 SDK 类型。

---

## 发现机制

Brigade 加载两组模块：

1. 随 Brigade 一起发布的**内置**模块。
2. **你的**模块，放入扩展文件夹：`~/.brigade/extensions/`。

该文件夹中的候选者要么是顶层文件（`*.js`、`*.mjs`、`*.ts`、`*.mts`），要么是包含 `index.{js,mjs,ts,mts}` 的文件夹。每个都必须 `export default` 一个模块（或模块数组）；其他情况会被跳过并给出明确原因——损坏的附加组件永远不会阻止 Brigade 启动。

TypeScript 直接加载：**没有构建步骤**。Brigade 在导入时转译 `.ts`/`.mts`，因此你可以用 TypeScript 编写并直接运行。

内置模块在 id 冲突时总是胜过你的模块，因此附加组件无法悄悄遮蔽核心能力。

---

## SDK 导入

你按名称导入 Brigade 稳定的编写接口——**不需要**将 Brigade 安装到扩展文件夹：

- `brigade/channel-sdk` — 用于编写**频道**（频道契约加上每个频道复用的共享辅助函数）。
- `brigade/extension-sdk` — 用于其他一切（完整的 `defineModule` + 能力接口）。

Brigade 会自动将这些导入解析到其自带的 SDK。

---

## 三种类型

`brigade extensions init <id> --kind <kind>` 为每种类型生成可运行的起始代码：

- **channel** — 将 Brigade 连接到消息平台。你实现一个小型适配器（`start` / `stop` / `sendText`，以及可选的媒体、表情回应、已读回执等），并用 `b.channel(adapter)` 注册。从 `brigade/channel-sdk` 导入。
- **tool** — 给智能体一个可在回合中调用的新动作。用 `b.tool({...})` 注册。从 `brigade/extension-sdk` 导入。
- **provider** — 接入外部能力（例如网页搜索后端）。用对应的 `b.*` 方法注册（`b.webSearch`、`b.webFetch`、`b.tts`、`b.mediaGen` 等）。提供方只有在报告自身已配置后才会激活。从 `brigade/extension-sdk` 导入。

默认类型是 `channel`。

---

## 安全模型

在非 Windows 系统上，Brigade 在加载每个候选文件**之前**对其应用安全检查，因此被篡改或放错位置的文件永远不会执行其代码：

- **全局可写**的文件被拒绝（任何人都能放入代码），
- 属于**其他非特权用户**的文件被拒绝，
- **指向扩展文件夹之外**的符号链接被拒绝。

每次拒绝都会报告原因并跳过该候选者；其余仍会加载。Windows 没有这些文件权限位，因此该门控在那里跳过。

还有两道护栏保护启动：顶层代码挂起的候选者会被限时并跳过，`register()` 挂起的模块也会被限时——行为异常的附加组件永远不会卡住回合或启动。

---

## CLI

### `brigade extensions list`

列出 Brigade 知道的所有扩展——内置的和你的——以及每个是否已加载。加 `--json` 获得机器可读输出。

### `brigade extensions doctor`

"我的插件为什么没加载"的排查界面。对于你的每个扩展，它逐步显示文件是否：

1. 通过了安全检查，
2. 加载无错误，
3. 导出了有效的 Brigade 模块。

出问题时它会准确告诉你哪一步失败以及如何修复。加 `--json` 获得机器可读输出。

### `brigade extensions init <id> [--kind channel|tool|provider]`

在 `~/.brigade/extensions/<id>/` 中生成起始扩展——一个接入正确 SDK 的 `index.ts` 加上简短的 `README.md`。如果同名文件夹已存在，它会（干净地）拒绝。编辑 `index.ts`，然后运行 `brigade extensions doctor` 确认加载。

---

## 最小频道

```ts
import { defineModule } from "brigade/channel-sdk";
import type { ChannelAdapter, ChannelStartContext } from "brigade/channel-sdk";

function createMyChannelAdapter(): ChannelAdapter {
  return {
    id: "my-channel",
    label: "My Channel",
    isConfigured(_cfg, _env) {
      return false; // return true once you have what you need to run
    },
    async start(ctx: ChannelStartContext) {
      // connect, and call ctx.onInbound({ channel: "my-channel", conversationId, from, text })
      // for each incoming message
    },
    async stop() {
      // tear down
    },
    async sendText(conversationId, text) {
      // deliver the reply
    },
  };
}

export default defineModule({
  id: "my-channel",
  register(b) {
    b.channel(createMyChannelAdapter());
  },
});
```

运行 `brigade extensions init my-channel` 即可为你生成这些（以及一个 README）。

---

## 频道槽位方法

`b.channel(adapter)` 注册连接工作器（`start` / `stop` / `sendText` / …）。另外三个上下文方法让频道可以接入中心接缝，而**不会**把其运行时拖入发送 / 入站 / 策略路径。每个都显式以 `channelId` 为键，从不调用它们的频道在构造上保持当前行为。

### `b.channelMessaging(channelId, adapter)` — 名称 / 句柄寻址

```ts
b.channelMessaging("my-channel", adapter);
```

注册一个 `ChannelMessagingAdapter`，使 `send_message` 工具能把智能体松散的 `to`（"Alex"、"@alex"、"my-channel:123"）转换为具体目标，并使按名称寻址的入站折叠到出站侧解析到的同一会话。

```ts
type ChannelMessagingAdapter = {
  // Required. Recognize an explicit `scheme:value` / `@handle` form → split into
  // an optional channel id + bare target. Return null when it isn't explicit.
  parseExplicitTarget: (text: string) => { channelId?: string; target: string } | null;
  // Required. Canonicalise a target id into the channel's stable shape. Must be idempotent.
  normalizeTarget: (raw: string) => string;
  // Optional. Resolve a human NAME/handle → a concrete id (a contact directory).
  // Return null (sync or async) when it doesn't resolve.
  targetResolver?: (name: string) => Promise<string | null> | string | null;
  // Optional. Best-effort DM-vs-group guess for a normalized target.
  inferTargetChatType?: (target: string) => "dm" | "group" | undefined;
  // Optional. Human-readable label for a target, for tool results / logs.
  formatTargetDisplay?: (target: string) => string;
  // Optional INBOUND hook (the inverse of targetResolver). Canonicalise an
  // incoming peer id → a stable conversation/session identity. Must be cheap,
  // side-effect-free, and must not throw.
  resolveInboundConversation?: (peerId: string) => string | null;
};
```

- **出站**通过 `resolveOutboundTarget({ channelId, to })` 运行：`parseExplicitTarget` → 可选的 `targetResolver`（仅对名称形状的输入）→ `normalizeTarget`。它**从不抛出**——行为异常的适配器会降级为原始 `to`。
- **入站**在路由解析器之前通过 `resolveInboundConversation({ channelId, peerId })` 运行；`null`/空结果（或无 Hook）保留原始 peer id，因此路由保持字节级一致。

### `b.channelSecurity(channelId, adapter)` — 仅收紧的 DM 策略咨询 + doctor 审计

```ts
b.channelSecurity("my-channel", adapter);
```

注册一个**补充性**的 `ChannelSecurityAdapter`。权威的访问控制引擎（配置 allow-from、所有者引导、配对）**不会**被替换——此适配器位于其之上。

```ts
type ChannelSecurityAdapter = {
  // Optional. A supplementary DM-policy opinion ("owner" | "allow-from" | "all"
  // | "disabled"), or null to take no opinion.
  resolveDmPolicy?: (ctx: ChannelSecurityContext) => ChannelSecurityDmPolicy | null;
  // Optional. Free-text warnings for `brigade doctor`.
  collectWarnings?: (ctx: ChannelSecurityContext) => string[] | Promise<string[]>;
  // Optional. Structured findings ({ checkId, severity, title, detail, remediation? })
  // for `brigade doctor`.
  collectAuditFindings?: (ctx) => ChannelSecurityAuditFinding[] | Promise<ChannelSecurityAuditFinding[]>;
};
```

- **DM 策略**通过 `consultChannelDmPolicy({ channelId, base, ctx })` 咨询，遵循严格的**仅收紧**规则：中心 `base` 策略保持权威，适配器只能将其在阶梯 `open < allowlist < pairing < disabled` 上移得*更紧*（作者值映射 `all→open`、`allow-from→allowlist`、`owner→pairing`、`disabled→disabled`）。会放松 `base` 的意见被忽略；`null`/抛出使 `base` 不变。它**从不抛出**。
- **审计**发现由 `collectChannelSecurityAudit(...)` 提供给 `brigade doctor` 的每频道安全部分；抛出的适配器被跳过，永不破坏 doctor 运行。

### `b.httpRoute(route)` — webhook 入站

```ts
b.httpRoute(route);
```

在网关的 HTTP 服务器上挂载一个 `HttpRoute`——推送传输频道（Telegram webhook、Slack Events API）或任何插件端点的入站路径。以配置为门控注册它，这样默认轮询安装不会暴露入站 HTTP 表面。

```ts
interface HttpRoute {
  method?: "GET" | "POST" | "PUT" | "PATCH" | "DELETE"; // any when omitted
  path: string;                                          // e.g. "/webhooks/my-channel"
  handler: (req: IncomingMessage, res: ServerResponse) => void | Promise<void>;
  // "none" (default): public — the handler verifies the provider's signature itself.
  // "operator": the gateway gates the route on operator-auth (admin endpoints).
  auth?: "none" | "operator";
  match?: "exact" | "prefix";   // "exact" (default); "prefix" matches sub-paths
  maxBodyBytes?: number;        // 413 above this; defaults to 1 MiB
  timeoutMs?: number;           // 408 past this; defaults to 30s
  skipSessionGuard?: boolean;   // opt out of the default-pass cross-agent guard
}
```

**首先**验证提供方的签名 / 密钥头（在解析 body 之前），然后将解析后的更新送入已启动适配器的规范化 + 去重 + 分发路径。**仅**当提供方通过处理程序自行校验的签名载荷进行认证时使用 `auth: "none"`（网关无法向第三方 webhook 出示 operator-auth）；否则使用 `auth: "operator"`。