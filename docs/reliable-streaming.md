# 可靠流式传输 — 网关订阅 + 渲染契约

这是 Web、移动端或桌面客户端流式接收 Brigade agent 输出（助手文本 + 交错的工具调用 + 工具结果）所遵循的契约，确保**消息始终按序到达、不丢失、每条都渲染在正确位置**——在弱网、手机后台、网关重启等场景下均如此，无需"下拉刷新"，也不会出现重复或错位的消息气泡。

参考客户端为 `src/tui/client.ts`（`BrigadeClient`）；参考渲染器为 `src/cli/commands/connect.ts`。新客户端在自己的传输层上复用相同的线格式（`src/protocol.ts`）。

## 核心思想

**已提交的对话记录是唯一事实来源。实时流只是其上的加速层。** 两者解析出相同的消息，以身份为键——因此任何丢失、乱序或重复的实时帧都能通过重读对话记录自愈，重复应用已有消息是空操作。

## 线格式帧（`src/protocol.ts`）

每个帧都是带 `type` 判别字段的 JSON：

| `type`     | 方向 | 含义 |
|------------|-----------|---------|
| `req`      | 客户端→服务端 | 请求：`{ type, id, method, params }` |
| `res`      | 服务端→客户端 | 配对回复：`{ type, id, ok, payload?, error?:{code,message,retryable?,retryAfterMs?} }` |
| `event`    | 服务端→客户端 | 推送：`{ type, event, payload, seq? }` |
| `tick`     | 服务端→客户端 | 保活 `{ type, ts }`（仅用于存活检测） |
| `shutdown` | 服务端→客户端 | 优雅重启通知 `{ type, reason }` |
| `hello-ok` | 服务端→客户端 | 连接后的**第一帧**（握手——见下文） |

流式 agent 输出以 `event: "pi"` 的 `event` 帧到达，其 `payload` 为 `{ event: PiEvent, agentId, sessionId, subagentDepth? }`。内层 `event` 符合导出的 `PiEvent` 判别联合（镜像 Pi 的 `AgentSessionEvent`；按 `event.type` 分支，取值见 `PI_EVENT_TYPES`）。`error.code` 为导出的 `ErrorCodes` 之一。

### 连接握手（`hello-ok`）

服务端在每条连接上发送的**第一帧**是 `hello-ok`——客户端读取它即可获知订阅所需的一切，无需硬编码：

```
{ type: "hello-ok",
  protocol: 1,                              // 仅在破坏性线格式变更时递增
  server: { version, connId, epoch },       // epoch = 进程启动 id（用于检测重启）
  features: { methods: string[], events: string[] },   // 所有可调用方法 + 可订阅事件
  policy: { maxPayload, maxBufferedBytes, tickIntervalMs },
  auth?: { role } }
```

`features.methods` 是实时列表（核心线格式方法 + 已注册的控制面 RPC）；`features.events` 是全部 `event` 名称。网关重启时 `epoch` 会变化——若与你上次看到的 epoch 不同，服务端的 seq 计数器已重置，因此**应作废你的 seq 游标并 `resume`**，而不是把新的低 seq 当作回退缺口。`BrigadeClient` 已为你处理（`"hello"` 事件 + `client.server`），并在 epoch 变化时自动清除游标。

### `seq` — 缺口检测器

每个**有序**帧都携带按会话单调递增的 `seq`（1、2、3、…）：顶层 `pi`、`approval-request` 和 `system-event`——它们**共享每个会话的一个计数器**，因此客户端能检测其中任一者的缺口。无序旁路通道（`state`、`error`、`log`）以及**子 agent 的 `pi` 帧**（`subagentDepth > 0`）**不带 `seq`**：子 agent 输出是子会话独立对话记录中的临时嵌套装饰，父会话的 `resume` 无法回填，因此实时渲染但不参与排序。可恢复的有序流是顶层对话 + 其审批 + 其系统通知。

客户端**按会话**跟踪最后看到的 `seq`。当下一个不是 `last + 1`——跳变（丢帧）、乱序或更低值（网关重启重置了计数器）——实时视图可能不完整，客户端应**恢复**（见下文）。会话中看到的第一帧永远不算缺口。客户端在 `resume` 时**不得**回退游标（游标由实时接收路径持有）；在繁忙会话上这样做会引发重同步风暴。（见 `src/protocol/stream-seq.ts`：`nextSeq`、`isSeqGap`。）

## `resume` — 恢复原语

```
req  { method: "resume", params: { agentId?, sessionKey? } }
res  { ok: true, payload: ResumeSnapshot }
```

`ResumeSnapshot = { sessionKey, agentId, messages, headSeq, pendingApprovals, recentSystemEvents, epoch, snapshot }`

- `messages` — 有序对话（`WireMessage[]`：`user` / `assistant` / `toolResult`），最旧在前。即已提交的对话记录。
- `headSeq` — 会话当前头部 `seq`。实时帧从 `headSeq + 1` 继续；任何 `seq ≤ headSeq` 的帧都是重复，幂等应用即可。（不要强行把游标设为此值——见上文 seq 说明。）
- `pendingApprovals` — 此会话上**仍待处理**的工具审批提示。这是针对唯一会丢失操作者*动作*的事件的恢复手段：在实时 `approval-request` 之后才连接或错过了它的客户端，渲染这些提示并通过 `approval-resolve` 解决，而不是让该轮次挂起至自动拒绝。无待处理项时为空。
- `recentSystemEvents` — 近期 `system-event` 通知的有界尾部，使离开期间错过通知的客户端仍能展示。
- `epoch` — 网关的进程启动 id；若自上次 `hello-ok` 以来已变化，说明服务端重启（作废游标）。
- `snapshot` — 头部状态（provider / model / tokens / running）。

`resume` 在文件系统和 Convex 存储模式下行为一致（读取 agent 持久化的同一份对话记录）。开销低，每次连接调用都安全。结合共享 seq，这意味着**已发出的内容不会丢失**：丢失的 `pi`/`approval-request`/`system-event` 帧会被检测到（seq 缺口）并在此完全恢复。

## 可靠性循环（客户端做什么）

1. **连接。** 打开 socket（仅当网关启用认证时发送 token——见 `docs`/`gateway-auth`）。第一帧是 `hello-ok`——读取协议版本、可调用方法 + 可订阅事件、策略限制和 `epoch`。将状态指示器绑定到客户端的 `"connection-state"` 事件（`connecting` / `connected` / `reconnecting` / `closed`）。
2. **订阅**目标通道：`req subscribe { agentId, sessionId }`。不订阅则接收全部（向后兼容）。

   **投递规则**，按顺序——首个匹配者生效：

   - 既无 `agentId` 也无 `sessionId` 的帧发给所有人。
   - `parentSessionKey` 为你所订阅会话的帧会被投递：你的线程派生的子 agent 归你可见。
   - **若你在帧的 agent 上指定了会话，那就是你的范围**——该 agent 的*其他*会话（cron 运行、频道流量、第二个聊天）会被过滤掉。传 `scope: "agent"` 可退出此行为，改为接收该 agent 的整个流；`scope: "session"` 为默认值。
   - 否则你会收到匹配更宽泛 `agentId` 或 `sessionId` 订阅的任何内容。

   过去指定会话会同时把投递范围扩大到整个 agent，导致其他线程渗入你的视图。若你的客户端指定会话是出于缩小范围之外的原因——例如以会话 id 为条件的快照推送——请传 `scope: "agent"`。

   **增量**为可选项：`subscribe { deltas: true }` 会从 `message_update` 帧中剥离 `message.content`（长回答下字节数约减少 18 倍），并要求客户端自行追加 `text_delta` / `thinking_delta`。不设置则帧与增量协议之前逐字节一致。

   两者均在 `hello-ok` 的 `features.capabilities` 中声明（`subscribe.scope`、`subscribe.deltas`、`resume.seq`）。**按这些名称做特性检测，而非按 `protocol`**——版本整数仅在帧本身变得不可解析时才递增，行为变更通过 capabilities 承载，因此未知名称可安全忽略。
3. **恢复。** `resume` → 渲染 `messages`（见渲染契约），将 seq 游标设为 `headSeq`。这会在首次连接时加载历史，在重连时回填缺口。
4. **实时流式应用。** 通过*同一个*渲染器应用每个 `pi` 帧。由于渲染以身份为键且幂等，与 resume 快照重叠的帧是无害的空操作。
5. **自动自愈。** 检测到 **seq 缺口**或**重连**时，回到步骤 3（`resume`）重建。永不丢失或卡住。

`BrigadeClient` 为你处理 seq 跟踪：缺口时发出 `"resync"` 事件并暴露 `resume()`（同时同步游标）。参考 TUI 在首次连接、`"reconnected"` 和 `"resync"` 时调用 `resume()`。

## 渲染契约（如何正确放置消息）

将对话记录建模为**以身份为键的有序块列表**，绝不按到达位置。通过更新带键块来应用增量；绝不盲目追加。

### 键

- **助手文本块：** `${subagentDepth}:${message.timestamp}`。
  Pi 在创建时为每条助手消息打上稳定的 `timestamp`，在该消息的所有 `message_update` 及其 `message_end` 中保持不变。工具调用**之后**的续写是**新**消息，带**新**时间戳 → 新块。`subagentDepth`（0 = 顶层）用于隔离子 agent 流。（辅助函数：`connect-transcript.ts` 中的 `asstKey`。）
- **工具块：** 工具调用的 `toolCallId`（从 `tool_execution_start` 到 `tool_execution_end` 保持稳定，并作为助手消息上的 `toolCall.id` 内容块及匹配的 `toolResult` 消息上的 `toolCallId` 出现）。

### 应用 Pi 事件

- `message_update` / `message_end` → `message` 是该助手消息的**完整累积快照**（非增量）。按键解析其块并**设置其内容**（替换式应用）。首次出现时创建块。
- `tool_execution_start` → 插入以 `toolCallId` 为键的工具块。**不要**关闭或重定位任何助手块——下一条助手消息带新时间戳，自然在工具下方开启新块。
- `tool_execution_end` → 更新 `toolCallId` 块，加上 ✓/✗ 和简短结果预览。
- **刷新规则：** 工具启动时，先绘制所有待处理的助手文本，使工具之前的文本渲染在其上方。

### 应用 resume 快照

按序遍历 `messages`，通过相同的带键应用渲染每条：
- `user` → 用户气泡。
- `assistant` → 其文本块（按上述方式键控）+ 每个 `toolCall` 内容块对应的待处理工具块。
- `toolResult` → 填充其 `toolCallId` 块的 ✓/✗ + 预览。

由于键与实时流匹配，在已有视图上重建的快照会就地更新块（无重复），仅追加缺失部分。参考 TUI 在 resume 时整体重建其对话记录区域；Web/移动客户端通常对带键消息列表做 diff（React/SwiftUI/Compose 使这变得简单）。

## 背压

网关限制每个客户端的发送缓冲区；落后过多的消费者会被关闭（`1008`）。这是**安全的**，因为客户端会重连并 `resume`——且由于 `seq` 在按客户端发送之前打戳，单个丢帧会使该客户端下一帧的 seq 跳变，同样触发 resume。任何丢弃都不会静默或永久。

## 心跳 / 存活检测

服务端目前的保活是周期性 `state` 广播（真实帧，会更新客户端的最后帧时间戳）加 WS 协议 ping；`tick` 帧类型为向前兼容而存在但尚未发出（客户端可容忍）。若在 `2 × TICK_INTERVAL_MS` 内无任何帧到达，客户端关闭并重连——捕获半开 socket，即常见的手机后台场景——然后 resume。

## 最小客户端伪代码

```ts
const client = new BrigadeClient({ url, token });
await client.connect();

const render = (piEvent) => applyByIdentity(transcript, piEvent); // 见契约
client.on("pi", (p) => render(p.event));

const recover = async () => {
  const snap = await client.resume({ agentId, sessionKey }); // 同步 seq 游标
  rebuildFromMessages(transcript, snap.messages);            // 带键、幂等
};

await client.request("subscribe", { agentId, sessionId: sessionKey });
await recover();                 // 初始历史
client.on("reconnected", recover); // 断开后回填
client.on("resync", recover);      // 流中缺口后回填
```

这就是整个契约：**订阅 → resume → 按身份应用实时帧 → 任何缺口再次 resume。** 按序、不丢失、不错位。

## 合成帧的游标重放

有序流在所有地方都可检测缺口，但仅在 JSONL 对话记录能重建的地方可修复。**合成**帧——Brigade 为 `claude-cli` 轮次铸造的工具事件，其工具在二进制自身的循环内运行——完全不在任何对话记录中，但它们携带路由会话键，因此 `resume` 能再次找到它们。

`FrameRing`（`src/core/frame-ring.ts`）按会话保留它们——受数量、字节数和不同会话数限制——使 `resume({ sinceSeq })` 返回 `replayedFrames`：最初广播的确切字节，在对话记录之后应用，并以与实时帧相同的身份键去重。TUI 从其 `resync` 处理器发送游标，此时最后看到的 seq 已在手。

顶层 `message_update` 帧刻意**不**保留：它们是累积的，缓冲长回复以重发对话记录已更好返回的内容会耗费 O(n²) 内存。

`replayComplete` 是契约中承重的部分。当保留已被裁剪至游标之前时它为 `false`，意味着某些帧已永久丢失，客户端应将该区间视为丢失而非假定为空。它只回答"我仍持有的最旧帧是否正是此游标期望的下一帧？"——从不回答"内部是否有缺口？"，因为造成那些缺口的帧恰恰是对话记录能更好重建的。

### 为何子 agent 帧仍不参与排序

它们看似同一情况，实则不然。子 agent 帧标记的是**子会话的 Pi session UUID**，而 `resume` 以 `agent:…` 会话**键**调用。在一个命名空间下保留、从另一个命名空间重放，意味着重放永远不会触发——因此对它们排序将依赖一个不存在的保证，这正是"绝不对无法重放的内容排序"规则所禁止的。

对它们排序还有该承诺本欲抵消的成本：每次 `spawn_agent` 都会铸造新的按会话计数器，而该映射按创建顺序驱逐，因此操作者自己的长生命周期会话——最先创建——最先被驱逐，导致其计数器重启，并在每个已连接客户端上重绘对话记录。终止性的 `agent_end` 帧还携带子会话的整个对话记录，环形缓冲的超大豁免会将其永久保留在一个再也不接收帧的会话上。

让子 agent 重放成真意味着教会 `resume` 子命名空间。在此之前它们保持不排序：其中的缺口是"不可检测但诚实"，而非"可检测但不可修复"。