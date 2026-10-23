---
name: taskflow-inbox-triage
description: 收件箱分诊的 TaskFlow 编写模式示例。当消息需要按意图区别处理时使用：部分路由立即通知，部分等待外部回复，其余归入稍后的汇总。
metadata: { "brigade": { "emoji": "📥" } }
---

# TaskFlow 收件箱分诊

这是一个具体示例，说明如何在不把核心运行时变成 DSL 的前提下设计 TaskFlow。

## 目标

用一个归属流程对收件箱条目分诊：

- 商务 → 发到 Slack 并等待回复
- 个人 → 立即通知归属者
- 其他 → 留到日终汇总

## 模式

1. 为收件箱批次创建一个流程。
2. 运行一个分离任务对新条目分类。
3. 将路由状态持久化到 `stateJson`。
4. 仅在需要外部回复时进入 `waiting`。
5. 分类或人工输入完成后恢复流程。
6. 批次路由完成后结束。

## 建议的 `stateJson` 结构

```json
{
  "businessThreads": [],
  "personalItems": [],
  "eodSummary": []
}
```

阻塞于 Slack 时建议的 `waitJson`：

```json
{
  "kind": "reply",
  "channel": "slack",
  "threadKey": "slack:thread-1"
}
```

## 最小运行时调用

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

const child = taskFlow.runTask({
  flowId: created.flowId,
  runtime: "acp",
  childSessionKey: "agent:main:subagent:classifier",
  task: "Classify inbox messages",
  status: "running",
  startedAt: Date.now(),
  lastEventAt: Date.now(),
});

if (!child.created) {
  throw new Error(child.reason);
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
  currentStep: "route_items",
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

## 相关示例

- `skills/taskflow/examples/inbox-triage.lobster`