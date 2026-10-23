# 🌊 Tideline — Brigade 的长期记忆引擎

Tideline 是支撑 Brigade 的长期记忆框架。它是一个**模型无关的记忆引擎**——零嵌入模型即可运行，提供模型则从中学习，并以 `brigade-tideline` 独立构建，与 Brigade 使用同一实现。

对话记录是 agent *刚说过的话*，Tideline 则是 agent *知道的事*：关于你和你的工作的持久事实，经信任门写入，按语义召回，过时则衰减，并随时间调和。

> TL;DR — 基于 JSONL 的事实，带来源作用域、溯源写入门、无需模型即可运行的混合关键词+向量召回、折叠为单一分数的双时态衰减、类型化链接图，以及每夜的反思/整合流程。一个 `Tideline` 门面；底层是一个小型适配器 SPI。

---

## 源码归属

`src/tideline/` 拥有可复用的记忆实现及其核心测试。`src/agents/memory/` 拥有 Brigade 的提取会话、行为审查、自动召回、扩展绑定和可选存储集成。旧模块路径是兼容性重导出（或薄宿主适配器），不是第二个引擎。

引擎分为 `api`、`store`、`ports`、`retrieval`、`graph`、`embeddings`、`extraction`、`lifecycle`、`governance`、`exports`、`transports/mcp` 和 `eval`。单元测试与各自模块放在一起；跨切面边界和端到端测试位于 `tests`。[源码地图](../src/tideline/README.md#source-layout)描述了各组的职责。三个公开包入口和 Brigade 的兼容导入在此内部重组中保持稳定。

当前原语包括记录与来源、源信任/写入门、召回分数、类型化链接、生命周期/保留、事件历史、可注入嵌入和评估用例。可选的 `FactStoreHostPorts` 为每个存储注入后端和日志器；Tideline 不需要 Convex 服务。

遗留的同步 `StorageAdapter` 和 JSONL 默认实现不是分布式数据库抽象。移动代码不声称带来任何新的基准、压缩或计费改进。

### 异步后端就绪与持久性

文件系统调用保持同步。使用 Brigade 的可选异步后端时，在首次操作前 await `store.ready()`（或 `memory.ready()`），然后在报告变更持久化前 await `store.flush()`（或 `memory.flush()`）。这两个方法对默认文件系统实现均为空操作。

```ts
await memory.ready();
memory.add({ content: "The staging window starts Friday.", segment: "project" });
await memory.flush();
```

冷同步读取现在报告待水合，而不是假装存储为空。水合失败是显式的；后续 `ready()` 会重试一个有界的获取周期。失败的写入保持待处理并使 `flush()` 拒绝；后续 flush 会重试它们，且不会重放已被取代的更新或删除。重试状态是进程本地的，不是崩溃持久的预写日志。事实刷新不会使尽力而为的事件追加与事实原子化，也不建立分布式缓存一致性。

Brigade 工具、提取、自动召回和计划维护使用这些屏障。MCP stdio 使用异步请求处理器，并在退出前排空待处理请求。自定义异步 MCP 传输必须使用 `handleAsync`，而非兼容的同步 `handle` 方法。

---

## 为什么存在

大多数"agent 记忆"只是一个向量存储加上相似度搜索。这在三个方面会失效，而 Tideline 正是为此而生：

1. **投毒。** 网页或工具结果绝不应能改写"操作者是素食者"。Tideline 的**写入门**拒绝不可信来源创作或取代受保护的事实。
2. **隐私泄漏。** 在多渠道团队中，WhatsApp 对端的事实绝不应出现在操作者的召回中。Tideline 按**来源**为每条记录划定作用域。
3. **过时与矛盾。** 信念会变。Tideline **衰减**未使用的事实，检测**矛盾**，并**整合**重复信念，而不是堆积重复项。

而且它在**不需要嵌入模型**的情况下完成这一切——召回运行在 BM25 主通道加上无模型向量*恢复*通道上，因此开箱即用，可离线、气隙环境运行。学习型嵌入器是可选的升级，绝不是依赖。

---

## 记录模型

每条记忆都是结构化记录，而非原始块：

| 字段 | 含义 |
|---|---|
| `content` | 事实，自然语言 |
| `segment` | `identity` · `preference` · `correction` · `relationship` · `project` · `knowledge` · `context` |
| `tier` | `short` · `long` · `permanent`（permanent 永不衰减） |
| `importance` | 0–1，调节衰减和排序 |
| `origin` | `owner`，或 `channel + conversationId + sessionKey` |
| `lifecycle` | `active` → `archived` → `pruned` |
| `links` | 指向其他记录的类型化边（见下方图） |

Segment 默认值（`SEGMENT_DEFAULTS`）为每个 segment 播种合理的 tier/importance——`identity` 和 `correction` 受保护且长期；`context` 是短期。

---

## 端到端：三个流程

### 1. 写入——门控并去重

```
add(fact) ──▶ content scan + write-gate ──▶ same-origin dedup ──▶ FactStore (JSONL)
              │
              └─ rejects an UNTRUSTED source (tool_output, retrieved_document,
                 extraction, compaction) trying to author/supersede a PROTECTED
                 segment (identity/preference/correction) → throws WriteGateError
```

**写入门**（`evaluateWriteGate`）对*来源*（可信：`user_instruction`、`owner_message`；不可信：其他一切）和*目标 segment* 进行分类。不可信写入受保护 segment 会被阻止；不可信写入其他位置会被**限制**（保留，但不允许覆盖可信信念）。去重**仅限同来源**——绝不跨主体合并。

### 2. 召回——混合、排序、来源作用域、预算化

```
origin + lifecycle filter ─▶ authorized candidates ─▶ BM25-primary + vector recovery
                                                              │
                                   optional graph recall ◀─────┘
                                              │
                                  ranked hits ─▶ context() budget block
```

- **混合通道**（`recallHybrid`）以 BM25 为主信号，以**无模型 HRR** 向量通道为恢复。HRR 捕获词汇/形态重叠，而非学习型同义。内置词袋嵌入器从召回查询中移除疑问框架，以避免仅功能词匹配；学习型嵌入器保留完整查询。存储的文档向量不变。学习型嵌入器可升级向量通道。
- **图召回**（`recallWithGraph`）沿类型化链接扩展命中，因此召回一条事实会拉入它 `supports` / `relates` / `supersedes` 的内容。
- **`effectiveScore`** 将双时态**衰减**（新近度 + 使用）和**源信任**加权折叠为单一排序数字。
- 存储检索在**排序前按当前调用的来源过滤候选**。底层评分器和图函数要求调用方提供已授权候选。`context()` 生成字符预算块，而非提供商 token 或计费测量。

### 3. 维护——衰减、dream、调和

- **衰减 GC**（`runDecayGc`）归档然后修剪被忽视的事实；`permanent` 免疫，已确认事实抵抗驱逐。
- **Dream**（`runDream`）是每夜的反思/整合/关联流程：确认重复信念、合并重复项、写入 `relates` 关联边，并驱逐衰减噪声。
- **矛盾检测**（`findContradictions`）暴露不一致的事实，以便更正取代过时的那条。

---

## 类型化链接图

事实不是无差别的袋子——它们是一个带**类型化边**的图：

`supersedes` · `transition` · `corrects` · `relates` · `derived_from` · `supports` · `contradicts`

这正是记忆得以*演化*的原因：更正 `supersedes` 旧信念，项目事实 `supports` 偏好，实体重命名是 `transition`。该图驱动图召回、矛盾处理，以及 **Memory Graph 仪表盘导出**（`exportMemoryGraph` → 节点、类型化边、通过确定性社区检测的主题聚类，以及标题统计）。

---

## 门面与适配器 SPI

上述一切通过一个对象访问：

```ts
import { Tideline } from "brigade-tideline";

const memory = Tideline.open("/path/to/workspace");

memory.add({ content: "I keep a strict vegetarian diet.", segment: "preference" });
const hits  = memory.recall("vegetarian diet");
const block = memory.context("vegetarian diet", { maxChars: 800 });
```

门面动词：**`add`**、**`recall`** / `search`、**`explain`**（某事实为何如此排序）、**`context`**（预算化提示块）、**`feedback`**（强化/惩罚），以及治理动词 **`purge`** / **`inspect`** / **`export`**。

Tideline 采用小型**适配器 SPI**，让你按自己的方式托管：

| 适配器 | 注入方式 | 用途 | v1 默认 |
|---|---|---|---|
| `StorageAdapter` | `Tideline.over(store)` | 持久化后端 | 内置 `FactStore`（JSONL） |
| `ClockAdapter` | `open`/`over` 选项 | 可注入时间 | 系统时钟 |
| `ThreatScanAdapter` | `open`/`over` 选项 | 召回时内容安全扫描 | 空操作（仅转义） |
| `EmbedderAdapter` | `open`/`over` 选项 | 学习型嵌入器接缝——**v1：保留** | 无模型 HRR |
| `LlmAdapter` | `open`/`over` 选项 | 反思/合成 LLM——**v1：保留** | 无 |

```ts
import { Tideline, FactStore } from "brigade-tideline";

const memory = Tideline.over(new FactStore(dir), {
  threatScan: { scan: (content) => myInjectionScanner(content) },
});
```

高级用户界面（`brigade-tideline/advanced`）直接暴露各流程——`runDream`、`runDecayGc`、`effectiveScore`、链接图（`buildGraph` / `neighbors` / `spread`）、治理（`purge` / `applyRetention` / `inspect` / `exportMemory`）、写入门（`evaluateWriteGate`）、透明性 `MemoryEventLog`，以及人工门控的自改进循环。

---

## 治理、透明性与自改进

- **治理**——`purge`（级联删除并清理链接）、`applyRetention`（策略驱动驱逐）、`inspect`（单条事实的溯源）、`exportMemory`。Brigade 通过 `manage_memory` 工具仅向所有者暴露这些（包括加密粉碎）。
- **透明性**——`MemoryEventLog` 为支持的记忆转换提供尽力而为的仅追加历史。它有助于解释已记录的变更，但不是完整或原子审计轨迹：事件写入可能独立于事实失败。
- **自改进循环**——*人工门控*循环：`proposeFromTelemetry` → `gateOnEval`（必须超过评估门槛）→ `approve` → `applyProposal` → `revertProposal`。召回行为的任何变更都必须通过评估和人工批准才能发布。

---

## 度量它（评估框架）

`brigade-tideline/eval` 是确定性评估框架——金标准集、带 bootstrap 置信区间的 **recall@k / MRR / nDCG@k**，以及基线和竞品能力用于头对头比较。

```ts
import { FactStore } from "brigade-tideline";
import { seedGold, RICH_GOLD, runRecallEval, hybridRecallCapability } from "brigade-tideline/eval";

const store  = new FactStore(tmpDir);
const cases  = seedGold(store, RICH_GOLD);
const result = await runRecallEval(hybridRecallCapability(store), cases, { k: 3 });
console.log(result.recallAtK, result.mrr, result.ndcgAtK);
```

你也可以在**自己的数据**上度量，而数据不离开本机：从真实事实导出金标准脚手架，将每个自动查询改写为现实释义，标记为 `approved`，然后运行。`loadGoldSpec` **拒绝**未批准的脚手架——人工改写才使其成为诚实度量，而非自匹配膨胀。

用 `npm run bench` 运行 Brigade 的内置基准。

---

## Brigade 如何使用 Tideline

- **工具：** `write_memory`（门控添加）、`recall_memory` / `read_memory`（跨事实 + `MEMORY.md` / `memory/*.md` 的混合搜索）、`manage_memory`（仅所有者治理：dream、shred、inspect、export、retention）。
- **自动召回：** 每轮之前，Brigade 注入来源匹配、预算化的上下文块——并对未知非所有者对端**失败关闭**，因此操作者记忆绝不泄漏到陌生人的会话。
- **后端：** 默认是文件系统 `FactStore`（同步 JSONL 读/改/写，面向遗留单操作者使用）。Brigade 的可选 Convex 集成提供每实例宿主端口；独立引擎不导入它。

---

## 打包状态

运行 `npm run build:tideline` 创建 `dist/packages/tideline/`，然后运行 `node scripts/test-tideline-package.mjs` 在隔离的临时消费者中检查。`npm run test:tideline-package` 执行这两项操作。

构建检查运行时和仅类型依赖闭包，发出严格声明，并在三个公开入口间共享 ESM 块。没有宿主模块替换或禁用的写入扫描器。Tideline 之外唯一允许的共享源码辅助是威胁模式、提示净化和原子重命名；它们不导入 Brigade 运行时。

构建不是发布。检查覆盖包行为和独立性；企业存储保证需要单独的合规和故障测试。独立输出与 Brigade 常规 `dist/tideline/` 编译输出分开，因此任一构建都可运行而不替换另一方的引擎文件。

---

## 另见

- 包清单与快速参考：[`src/tideline/README.md`](../src/tideline/README.md)
- 公开 API 表面：[`src/tideline/index.ts`](../src/tideline/index.ts) ·
  高级：[`src/tideline/advanced.ts`](../src/tideline/advanced.ts)
- 引擎实现：[`src/tideline/`](../src/tideline/)
- Brigade 集成与兼容性：[`src/agents/memory/`](../src/agents/memory/)
- 产品中的记忆：[README → Features → Memory](../README.md#-memory)

_许可证：MIT。_