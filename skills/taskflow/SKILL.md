---
name: taskflow
description: 当工作需跨越一个或多个分离任务，但仍应表现为由单一所有者上下文驱动的同一作业时使用。TaskFlow 是 Lobster、ACPX、插件或普通代码等编写层之下的持久化流程基座。条件逻辑保留在调用方；TaskFlow 负责流程标识、子任务关联、等待状态、带修订检查的变更以及面向用户的呈现。
metadata: { "brigade": { "emoji": "🪝" } }
---

# TaskFlow

当作业需要比单次提示或单次分离运行存活更久，但仍需一个所有者会话、一个返回上下文和一个检查或恢复工作的位置时，使用 TaskFlow。

## 何时使用

- 由单一所有者驱动的多步骤后台工作
- 等待分离的 ACP 或子代理任务的工作
- 可能需要向所有者发出一次清晰更新的作业
- 需要在步骤之间持久化少量状态的作业
- 必须在重启和修订冲突中干净存活的插件或工具工作

## TaskFlow 负责的内容

- 流程标识
- 所有者会话和请求方来源
- `currentStep`、`stateJson` 和 `waitJson`
- 关联的子任务及其父流程 ID
- 完成、失败、取消、等待和阻塞状态
- 用于冲突安全变更的修订跟踪

它**不**负责分支或业务逻辑。将这些放在 Lobster、acpx 或调用代码中。

## 当前运行时形态

规范的插件/运行时入口点：

- `api.runtime.tasks.flow`
- `api.runtime.taskFlow` 仍作为别名存在，但 `api.runtime.tasks.flow` 是规范形态

绑定：

- 当你已有带 `sessionKey` 的可信工具上下文时，使用 `api.runtime.tasks.flow.fromToolContext(ctx)`
- 当绑定层已解析会话和投递上下文时，使用 `api.runtime.tasks.flow.bindSession({ sessionKey, requesterOrigin })`

托管流程生命周期：

1. `createManaged(...)`
2. `runTask(...)`
3. 等待人员或外部系统时调用 `setWaiting(...)`
4. 工作可继续时调用 `resume(...)`
5. `finish(...)` 或 `fail(...)`
6. 整个作业应停止时调用 `requestCancel(...)` 或 `cancel(...)`

## 设计约束

- 当你的代码负责编排时，使用**托管** TaskFlow。
- 单任务**镜像**流程由核心运行时为分离的 ACP/子代理工作创建；本技能主要涉及托管流程。
- 将 `stateJson` 视为持久化状态包。没有单独的 `setFlowOutput` 或 `appendFlowOutput` API。
- 创建后的每个变更方法都会进行修订检查。每次成功变更后携带最新的 `flow.revision`。
- `runTask(...)` 将子任务关联到流程。需要父级编排时，用它代替手动创建分离任务。

## 示例形态

```ts
const taskFlow = api.runtime.tasks.flow.fromToolContext(ctx);

const created = taskFlow.createManaged({
  controllerId: "my-plugin/inbox-triage",
  goal: "triage inbox",
  currentStep: "classify",
  stateJson: {
    businessThreads: [],
    personalItems: [],
    eodSummary: [],
  },
});

const classify = taskFlow.runTask({
  flowId: created.flowId,
  runtime: "acp",
  childSessionKey: "agent:main:subagent:classifier",
  runId: "inbox-classify-1",
  task: "Classify inbox messages",
  status: "running",
  startedAt: Date.now(),
  lastEventAt: Date.now(),
});

if (!classify.created) {
  throw new Error(classify.reason);
}

const waiting = taskFlow.setWaiting({
  flowId: created.flowId,
  expectedRevision: created.revision,
  currentStep: "await_business_reply",
  stateJson: {
    businessThreads: ["slack:thread-1"],
    personalItems: [],
    eodSummary: [],
  },
  waitJson: {
    kind: "reply",
    channel: "slack",
    threadKey: "slack:thread-1",
  },
});

if (!waiting.applied) {
  throw new Error(waiting.code);
}

const resumed = taskFlow.resume({
  flowId: waiting.flow.flowId,
  expectedRevision: waiting.flow.revision,
  status: "running",
  currentStep: "finalize",
  stateJson: waiting.flow.stateJson,
});

if (!resumed.applied) {
  throw new Error(resumed.code);
}

taskFlow.finish({
  flowId: resumed.flow.flowId,
  expectedRevision: resumed.flow.revision,
  stateJson: resumed.flow.stateJson,
});
```

## 条件逻辑保持在运行时之上

使用流程运行时处理状态和任务关联。决策保留在编写层：

- `business` → 发布到 Slack 并等待
- `personal` → 立即通知所有者
- `later` → 追加到日终摘要桶

## 操作模式

- 只存储恢复所需的最小状态。
- 将人类可读的等待原因放入 `blockedSummary`，或将结构化等待元数据放入 `waitJson`。
- 当编排器需要子工作的紧凑健康视图时，使用 `getTaskSummary(flowId)`。
- 当调用方希望流程立即停止调度时，使用 `requestCancel(...)`。
- 当还需取消活跃的关联子任务时，使用 `cancel(...)`。

## 示例

- 参见 `skills/taskflow/examples/inbox-triage.lobster`
- 参见 `skills/taskflow/examples/pr-intake.lobster`
- 参见 `skills/taskflow-inbox-triage/SKILL.md` 了解具体路由模式