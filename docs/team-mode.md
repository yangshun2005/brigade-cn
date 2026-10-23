# Team Mode

Team Mode 是 Brigade 的持久协作层，面向需要多个 agent 协作的工作。它把用户目标转化为一个房间范围内的运行，其中包含任务图；通过 Brigade 现有的 agent 循环执行就绪任务，并将每个完成的结果返回图中，以便下游 agent 继续推进。

它属于 Brigade，而非 Tideline。它可在两种 Brigade 存储模式下运行：

- **filesystem**（默认）：Brigade 状态目录下带锁、哈希链、fsync 的日志；
- **Convex**：规范化的协作表，每个语义命令一个乐观并发事务。

使用 `brigade store migrate` 切换存储模式时，Team 房间、运行、消息、任务、尝试、交接、审批、构建产物、事件、outbox 行、命令回执和房间游标会随其余 Brigade 状态一起迁移。

## 房间会话模型

房间是持久的群组/频道边界。它拥有标题、配置的 agent 成员、公开会话、运行历史和权限。房间本身不是运行或记录：一个房间可随时间容纳多个会话和 Team 运行。线程是持久房间消息之间的关系，因此回复保留相同的房间权限并重连历史，而不会变成隐藏的 agent 会话。

公开房间消息支持回复和线程、显式 `@agent` 提及、附件引用、编辑和软删除历史、表情回应、置顶、有界搜索、分页和权威房间指标。提及用于寻址成员并帮助 agent 找到上下文；它们从不静默创建工作。操作是显式的：

- **消息或提及**：在公开房间中通信；
- **分配/委派**：创建由 agent 拥有的持久工作；
- **咨询**：询问子 agent，然后用其结果恢复调用方；
- **交接**：在接受后转移当前任务的所有权。

房间协调者的可见回复会自动持久化为公开消息，包括重启后恢复的隐藏完成唤醒。租约 worker 可读取公开房间并发布带 fence 的尝试更新。浏览器只能以已认证操作员身份发布；不能冒充 agent，也不能绕过 worker 的活动租约和 fence。

## 执行模型

最终审查任务可声明固定的 `resultGate: { "kind": "review_verdict" }` 契约。worker 在可信执行信封中收到该要求，共享权威只接受以 `REVIEW: PASS` 开头的结果。`REVIEW: FAIL`、模糊输出和非字符串结果会在 filesystem 和 Convex 两种存储模式下以 `RESULT_GATE_FAILED` 原子地使尝试、任务和运行失败。

协商 `team.review-policy.independent-v1` 的客户端可选择更强的 `resultGate: { "kind": "review_verdict", "policy": "independent-v1" }` 契约。权威要求显式分配的审查者、一个或多个带 `all` 连接的直接依赖，以及每个直接、传递和动态委派的上游贡献者对应不同的显式 agent。它在运行开始、交接接受和审查完成时重新检查持久化的尝试身份。通过的回复必须首行为 `REVIEW: PASS`，第二行精确为 `EVIDENCE: {"task-id":"sha256"}`，包含每个直接依赖且不含其他键。摘要来自无损结果分页 API。省略 `policy` 会为现有客户端和持久运行刻意保留原始的仅判定行为。

运行状态通过截断长结果预览保持紧凑。协调者可用 `team({ action: "read_result", runId, taskId, offset, limit })` 分页读取一个无损结果，网关客户端可用 `team.tasks.result`。租约 worker 通过 `team_task` 使用相同的分页契约，但只能读取其直接依赖；返回任何内容前会检查尝试租约/fence 和房间/运行边。每页携带完整编码结果的 SHA-256 和 `nextOffset`，直到 `complete` 为 true。

```text
room
  └─ run (objective + budgets)
       ├─ task A ───────────────┐
       ├─ task B ───────────────┼─> task D (all / any / quorum join)
       └─ task C ───────────────┘
            └─ attempt (lease + fence + runtime turn)
```

存储是权威。worker 必须在运行 agent 轮次前原子地认领就绪任务。每次认领获得租约令牌和单调递增的 fence；租约被替换后，过期 worker 不能完成、失败、交接、审批或附加构建产物。网关续期活动租约，重启时协调过期工作，并在持久取消后中止匹配的 Brigade 轮次。

任务支持：

- `all`、`any` 和法定人数依赖连接；
- 在 `any` 或法定人数连接后可选取消失败分支；
- 有界尝试、重试过滤器和重试退避；
- 操作员显式重试失败任务，仅在房间没有更新的活动运行时安全重开其失败运行和依赖跳过的后代；
- 每次运行的 token、成本、耗时、并发和尝试预算（若提供方无法报告完整定价，成本上限会失败关闭）；
- 持久审批、接受/拒绝的交接和尝试范围的构建产物；
- 尝试范围的 worker 记录，使用独立的稳定房间加 agent 预留键，使重试/任务不会静默相互污染，同时房间内同一 agent 的工作保持串行。

权威认领调度器的确切任务，并在尝试中持久化其解析的 agent 身份。该事务级检查防止另一运行时或过期快照将已分配认领重定向到未分配任务，或并发运行同一 agent 工作区。审批和交接请求默认五分钟后过期，上限 24 小时。终态发布等待提供方用量归因；外部取消或已交接的回调获得有界 30 秒结算宽限期，之后重启协调将其用量记录为未知。

agent 的正常最终回复成为其任务尝试的持久结果。依赖结果和构建产物被插入下一任务的执行信封。这条完成路径使多 agent 工作汇聚回运行，而非以断开的子会话结束。

当运行达到终态，持久 outbox 在房间协调者独立的 `team-chat` 会话中启动一个隐藏的可信轮次，以确切事件通知为输入。它不使用共享聊天收件箱，因此操作员消息不能消费或重排完成事件。协调者用 `team status` 读取权威结果并为操作员综合。该返回路径由运行时拥有；不依赖 worker 记得调用消息工具。这些唤醒轮次标记为内部记录：可触发模型，但不能渲染为操作员撰写的聊天。失败的唤醒留在 outbox 中重试，而非被确认并丢失。若配置的协调者已被移除，运行时和工具授权路径选择同一活动房间成员，而非创建脑裂会话。Outbox 投递与调度分开运行，因此慢协调者模型或传输不能阻塞网关启动、任务认领、取消或租约工作。唤醒协调者前，过期的终态和决策行会对照当前权威状态检查，因此重试的运行不会被后续叙述为失败。

## Agent 工具

面向所有者的 `team` 工具创建、更新和安全归档房间；创建运行；添加任务图；启动或取消工作；检查状态（包括有界持久结果）；重试任务；解决待审批和交接。它还可列出、搜索和发布公开房间消息。房间更新可更改标题或配置成员，应用补丁时保留现有元数据，拒绝未知 agent，且不能移除仍拥有活动工作的 agent。在 `team-chat` 会话中，房间操作默认且限制为该房间。首选会话调用是 `delegate`，它在单个幂等权威事务中验证并启动完整 DAG（线上为 `team.runs.delegate`）。无效图不会留下部分运行、事件、outbox 行或命令回执。较低级序列仍可用于分阶段或异常大的图：

1. 用参与的 Brigade agent ID `create_room`。
2. 用目标和可选预算 `create_run`。
3. 用稳定任务 ID、分配、依赖和连接 `add_tasks`。
4. `start_run`。
5. 用 `status` 检查持久进度并回答待决策。

执行租约任务时，agent 仅获得工作区执行工具（`read`、`write`、`edit`、`bash`、`grep`、`find`、`ls`）、只读研究工具（`web_search`、配置时的 `fetch_url`）和窄化的 `team_task` 工具。它可检查状态、请求并等待审批、提供并等待所有权交接、注册构建产物，或原子地委派一至四个子任务。它还可读取房间消息并发布与其尝试绑定的带 fence 公开更新。委派不同于交接：源尝试让出租约，父任务进入 `waiting_children`，运行时仅在每个直接子任务终态且源用量持久结算后启动新的带 fence 父尝试。恢复的提示包含子结果、失败和构建产物。请求键使重试语义幂等；谱系、嵌套深度、总子数、成员资格、循环和运行预算在提交任何子任务前经权威检查。租约令牌、所有者和 fence 保存在执行范围的能力中，从不暴露给模型或网关运行快照。所有者/全局配置和凭据管理、记忆、跨会话、交互式浏览器、其他扩展和通用子 agent 工具从 Team worker 轮次中移除。尝试记录对普通会话列表/历史/发送/变更 API 隐藏；授权检查通过 Team 运行状态进行。Team 轮次还使用非所有者记忆来源，因此自动所有者记忆召回和轮次后所有者记忆提取保持禁用。

worker 间提问使用相同返回路径而非第二个有损聊天通道：调用方用 `delegationKind: "consultation"` 委派一个子任务，让出当前尝试，并用被咨询成员的持久结果恢复。这使“询问此 agent 并回复我”重启安全、用量可计、可递归嵌套并在任务谱系中可见。仅当接收成员应永久拥有原任务时才使用交接。

此能力收窄不是硬操作系统沙箱。Team worker 当前是可信本地 Brigade agent：文件工具遵循 Brigade 普通工作区策略，批准的 shell 命令以网关进程的主机访问运行。不要在此阶段于一个网关中执行互不信任的租户。SaaS 边界需要隔离的 worker 进程或容器、任务范围挂载、凭据代理和后续多租户控制平面中的出口策略。

## 房间 UI 的网关 API

握手通告 `team.*` 方法和 Team 能力。相关方法包括：

```text
team.rooms.list/create/update/archive/metrics
team.messages.list/search/post/edit/delete/react/pin
team.runs.list/create/delegate/start/get/cancel
team.tasks.list/result/add/cancel/retry
team.handoffs.list/respond
team.approvals.list/resolve
team.artifacts.list
team.events.list
team.resume
```

失败请求保留网关的结构化错误信封（`code`、`message`、`retryable`、可选 `retryAfterMs` 和 `details`）。Team 客户端可基于 `TEAM_NOT_FOUND`、`TEAM_CONFLICT`、`TEAM_BUDGET_EXHAUSTED` 和 `TEAM_DOMAIN_ERROR` 分支；无效输入继续使用 `INVALID_REQUEST`。当前 Team 域失败标记为不可重试，因此客户端应刷新状态或更改命令，而非盲目重放。

连接后显式订阅；Team 事件从不进入旧版 agent/会话 firehose：

```json
{
  "type": "req",
  "id": "sub-1",
  "method": "subscribe",
  "params": { "roomId": "room-id", "includeProgress": true }
}
```

基于 `team.subscribe.snapshot` 能力进行重连安全的水合。对于 `subscribe { roomId }`，网关先安装活动房间路由，然后返回该房间的权威 `TeamRoomListSummary` 作为响应载荷。立即水合它：安装前提交的任何内容都在摘要中，之后提交的任何内容有资格进入活动事件通道。通过 `headRoomSeq` 和 `execApprovalRevision` 协调延迟摘要，因为它可能在更新的活动帧之后到达。仅旧版 agent/会话订阅保留无载荷的成功响应。

有四条事件通道：

- `team-event` 是持久生命周期转换。`roomSeq` 每房间严格单调且可重放。公开消息发布、编辑、删除、表情回应和置顶也使用此通道，因此每个选定房间的客户端都会收敛。
- `team-progress` 是活动 assistant/工具/心跳装饰。仅在尝试内按 `progressSeq` 排序，背压下可丢弃，从不推进持久房间游标。工具参数和结果从此通道省略，以避免泄露凭据或文档内容。
- `team-approval-request` 仅将 worker shell 审批路由到其房间的订阅者。`team-approval-resolved` 为每个订阅客户端关闭它。两个事件和 `team.resume` 都携带单调递增的进程本地审批修订，因此延迟的恢复快照不能擦除更新的请求或复活已解决的提示。`team.resume` 还包含每个仍待处理的运行时审批，因此重连不会让活动 worker 在不可见提示后等待。
- 原始 worker Pi 帧是私有的，不广播，包括对无订阅的旧版客户端。因此 worker 分配信封不能出现在操作员会话中；UI 仅接收脱敏的 Team 通道。

重连后，用最后渲染的 `roomSeq` 调用 `team.resume`。响应包含当前房间/运行快照、权威指标、最近 100 条公开消息和更新的持久事件。用 `team.messages.list` 分页更早会话。优先使用 `beforeMessageId`/`afterMessageId` 而非旧版时间戳游标，以免同一毫秒创建的消息被跳过；游标必须命名同一房间中的消息。使用 `team.messages.search` 进行有界服务器端搜索。若 `replayComplete` 为 false，将返回的快照渲染为权威，而非从完整事件区间重建状态。

对于 UI 状态，按 `attemptId` 键控瞬态流，从 `team-event`/`team.resume` 渲染持久任务状态，并将 `task.succeeded`、`task.failed`、`task.handed_off` 或 `task.cancelled` 视为任何进行中装饰的终态替换。

Brigade 有意将其作为无头协作后端发布。产品客户端应基于版本化方法和协商能力构建，在进入协调者有序 `prompt` 通道前提交操作员消息，通过稳定消息 ID 协调乐观行，并防止仅线程回复静默启动工作。为协调者会话使用稳定的房间范围 `team-chat` 会话；独立的 `team` 尝试命名空间使 worker 分配提示远离该记录。订阅每个房间以获取轻量生命周期更新，仅为选定房间启用进度，并使用 `team.resume` 游标和修订协调重连。不要从配置推断在线状态，不要在流式传输时强制滚动阅读器，也不要在按时间顺序的房间上下文之外隐藏待审批和交接决策。

隐藏完成和决策轮次在代码中限制为一个只读 `team` 状态工具，并跳过轮次后记忆提取。该确切合成形状可绕过工作区准入，使暂停在 Team 任务内的协调者能呈现自己的待审批而不会自死锁；任何普通或具备工作区能力的轮次都不会获得该绕过。

## 可靠性边界

Filesystem 模式面向单个 Brigade 网关，通过其日志在进程崩溃后存活。Convex 模式通过现有可选后端提供相同的协作语义。运行时仍是单网关：并发和每会话预留是进程本地的，因此多个网关不得并发执行同一所有者的运行。

Convex Team Mode 还是**单所有者 Phase-1 规模配置**，尚不是无界企业历史存储。一次原子委派在任何权威状态变更前上限为 512 KiB，为其持久命令回执在 Convex 每文档限制下留出余量。更重要的是，当前 Convex 适配器在每次语义变更前在一个查询中水合所有者的所有房间、运行、任务、尝试、交接、审批、构建产物和房间游标。因此保留的核心状态必须低于 Convex 的 16 MiB 事务读取限制。更大保留历史请使用 filesystem 模式；较低级分阶段图 API 不会移除该累积 Convex 限制。

移除该限制需要修订 fence 的分页水合变更，而不仅是更大的常量：

1. 在 `convex/collaboration.ts` 中为每个规范化协作集合添加轻量协作修订查询和游标分页查询。按记录数和编码字节双重限制页大小。
2. 在 `src/storage/convex/collaboration-store.ts` 中，读取修订，水合所有必需页，然后再次读取修订。若变化则重试读取，使内存语义存储永不接收撕裂的跨页快照。
3. 在语义评估后保留现有预期修订提交 fence、有界命令增量和原子回执/事件/outbox 提交。
4. 通过同一分页读取器路由快照导出、精确迁移验证和重启水合。添加超过 16 MiB 保留状态的实时 Convex 测试，以及页间竞争的变更，然后移除 Phase-1 标签。

两种模式都不使 Brigade 成为多租户服务，可选 Convex 函数也不是租户授权边界。SaaS 身份、租户限定授权、配额、集群范围准入和网络隔离仍是独立平台层。这使 Team Mode 可在本地或本地部署完成和使用，而不与后续 SaaS 控制平面耦合。