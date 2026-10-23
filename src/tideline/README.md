# brigade-tideline

面向 agent 的模型无关**长期记忆引擎**——支撑 Brigade 记忆的框架，封装在单一门面之后。

- **混合召回**——以 BM25 为主，辅以无模型 HRR 向量*恢复*通道（运行无需嵌入模型；学习型嵌入器是可选升级，不是依赖）。
- **双时间衰减 + 信任调制**——新近度/使用衰减与来源信任权重合并为一个有效分数。
- **来源写入门禁**——不可信来源（工具输出、检索文档、蒸馏提取）不能创建或取代操作者的身份/偏好/纠正；投毒写入会被拒绝。
- **按来源隔离**——所有者事实与按频道/对等方事实分别限定作用域，一个主体的记忆不会泄漏到另一个主体的召回中。
- **类型化链接图**——`supersedes` / `transition` / `corrects` / `relates` / `derived_from` / `supports` / `contradicts` 边，支持图召回遍历。
- **反思 / 整合 / 关联**——夜间任务确认重复信念、合并重复项、持久化 `relates` 关联边，并驱逐衰减噪声。
- **评估工具**（`brigade-tideline/eval`）——确定性黄金集、带 bootstrap 置信区间的 recall@k / MRR / nDCG@k、基线 + 竞品能力用于对比，以及隐私安全的导出→审批流水线，用于在自有数据上测量。

## 本地构建

```
npm run build:tideline
npm run test:tideline-package
```

## 快速开始

```ts
import { Tideline } from "brigade-tideline";

const memory = Tideline.open("/path/to/workspace");

// 写入（应用写入门禁 + 去重）。
memory.add({ content: "I keep a strict vegetarian diet.", segment: "preference" });

// 使用词法重叠召回；无模型向量不学习同义关系。
const hits = memory.recall("vegetarian diet");

// 预算受限、按来源限定作用域的块，可直接放入提示词。
const block = memory.context("vegetarian diet", { maxChars: 800 });
```

### 适配器 SPI

`Tideline.open(dir, opts)` 接受 `TidelineAdapters`；同步的
`StorageAdapter` 也可通过 `Tideline.over(store, opts)` 提供。
可选的每实例 `hostPorts` 提供旧存储的后端和日志器。
不提供时，引擎使用本地 JSONL，不依赖 Brigade 运行时：

对于异步快照后端，在读写前 await `memory.ready()`，在确认持久化前 await `memory.flush()`。两者都转发到可选适配器方法，对文件系统存储为空操作。Brigade 的后端会显式报告待处理/失败的水合，并拒绝失败的 flush；后续的 readiness/flush 调用会重试保留的工作。待处理重试是进程本地的，不具崩溃持久性，事实 flush 不包含尽力而为的审计事件。MCP stdio 使用 `handleAsync` 来强制执行这些屏障；同步 `handle` 方法保留给已就绪的同步消费者。

| 适配器 | 注入方式 | 用途 | v1 默认 |
|---|---|---|---|
| `StorageAdapter` | `.over(store)` | 持久化后端 | 内置 `FactStore`（文件系统 JSONL） |
| `ClockAdapter` | `.open`/`.over` 选项 | 可注入时间 | 系统时钟 |
| `ThreatScanAdapter` | `.open`/`.over` 选项 | 召回时内容安全扫描 | 空操作（仅标记转义） |
| `EmbedderAdapter` | `.open`/`.over` 选项 | 学习型嵌入器接缝——**v1：保留**，已记录但尚未调用（召回始终使用内置 HRR 通道） | 无（无模型 HRR） |
| `LlmAdapter` | `.open`/`.over` 选项 | 反思/合成 LLM——**v1：保留**，未使用 | 无 |
| `FactStoreHostPorts` | `.open` 选项 / `new FactStore(dir, { hostPorts })` | 可选后端和日志器 | 本地文件系统，空操作日志器 |

```ts
import { Tideline, FactStore } from "brigade-tideline";

const memory = Tideline.over(new FactStore(dir), {
  threatScan: { scan: (content) => myInjectionScanner(content) },
});
```

### 评估

```ts
import { FactStore } from "brigade-tideline";
import { seedGold, RICH_GOLD, runRecallEval, hybridRecallCapability } from "brigade-tideline/eval";

const store = new FactStore(tmpDir);
const cases = seedGold(store, RICH_GOLD);
const result = await runRecallEval(hybridRecallCapability(store), cases, { k: 3 });
console.log(result.recallAtK, result.mrr, result.ndcgAtK);
```

在**自有数据**上测量（隐私安全——真实事实永不离开本机）：

```ts
import { FactStore } from "brigade-tideline";
import { exportGoldScaffold, writeLocalGoldSpec, loadGoldSpec, seedGold } from "brigade-tideline/eval";

writeLocalGoldSpec("gold.local.json", exportGoldScaffold(new FactStore(realDir)));
// → 审阅 gold.local.json：将每个自动查询改写为真实的转述，
//   设置其分类类别，然后设置 "approved": true。
const cases = seedGold(new FactStore(tmp), loadGoldSpec("gold.local.json"));
```

`loadGoldSpec` **拒绝**未批准的脚手架（其自动查询会自匹配自身事实，从而虚高召回）——人工审批改写才使其成为真实测量。

### 高级（`brigade-tideline/advanced`）

门面所构建于其上的高级用户接口：生命周期过程（`runDream`、`runDecayGc`、`effectiveScore`）、类型化链接图（`buildGraph` / `neighbors` / `spread` / `synonymyEdges` / `resolveEntities`）、治理（`purge` / `applyRetention` / `inspect` / `exportMemory`）、来源写入门禁（`evaluateWriteGate` + 信任/分段辅助函数）、透明度事件日志（`MemoryEventLog`），以及人工门控的自改进循环（`proposeFromTelemetry` → `gateOnEval` → `approve` → `applyProposal` → `revertProposal`）。`WriteGateError`（由 `Tideline.add` 抛出）也在**主**入口上，调用方无需访问 `/advanced` 即可捕获。

## 打包状态

### 源码布局

```text
src/tideline/
  index.ts / advanced.ts / eval.ts   稳定的公共包入口
  api/                              Tideline 门面和适配器 API
  store/                            记录、JSONL 存储和事件历史
  ports/                            可选后端和日志契约
  retrieval/                        查询、评分、混合和图召回
  graph/                            类型化链接、图操作和投影
  embeddings/                       嵌入器、提供者和重新嵌入
  extraction/                       解析、关系提取和 LLM 回调
  lifecycle/                        衰减、策展、反思和维护
  governance/                       写入门禁、保留、检查和清除
  exports/                          文件系统保险库投影
  transports/mcp/                   MCP 工具、JSON-RPC 服务器和 stdio 传输
  eval/                             评估工具、指标和夹具
  tests/                            公共 API、隔离、边界和端到端测试
```

单元测试与其实现放在一起。横切测试位于
`tests/`；Brigade 宿主集成测试保留在 `src/agents/memory/`，实际 CLI 测试保留在 `src/cli/commands/`。

内部模块导入其特定依赖，而非公共入口桶。核心模块不导入 MCP 传输、评估或测试代码。传输和评估依赖引擎，Brigade 从外部提供宿主端口。边界测试对运行时和仅类型导入强制执行这些规则。这些是所有权边界，不是独立服务，也不断言旧依赖图无环。

### 构建与兼容性

本目录拥有规范的可复用引擎和核心测试。Brigade 的运行时集成导入它；`src/agents/memory/` 中的旧路径是兼容导出或薄宿主适配器，不是另一个引擎。Agent 会话、调度、自动召回和扩展注册保留在 Brigade 中。

`npm run build:tideline` 构建 `dist/packages/tideline/`，包含三个公共入口（`.`、`/advanced`、`/eval`）、严格声明和共享 ESM 块。这与 Brigade 正常的 `dist/tideline/` 编译器输出分开。构建拒绝引擎之外的运行时和仅类型依赖，除了三个经审查的共享辅助函数：威胁扫描、提示词净化和原子重命名。

没有强制性的 Convex 包、宿主模块替换或禁用的写入扫描器。Brigade 通过 `FactStoreHostPorts` 提供其可选的缓存后端存储。`npm run test:tideline-package` 检查隔离消费者、跨入口模块身份、来源隔离、扫描器一致性、持久化、现有评估夹具和声明解析。本地构建不会发布包。

## 本次重构的范围

现有原语包括事实/来源、来源信任和写入门禁、排序检索、类型化链接、生命周期/保留、事件历史、嵌入和评估。移动所有权保留其行为；它本身不会改善基准。字符预算上下文不是 token 或计费节省的测量。

内置存储仍使用同步 JSONL 读/改/写。可替换的旧端口不提供分布式事务、租户权威、完整影响捕获、即时撤回屏障或认证的企业存储适配器。独立记忆包和 Brigade 工作区使用相同实现，并保留这些相同的旧限制。

## 许可证

MIT。