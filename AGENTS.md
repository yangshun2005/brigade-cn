# AGENTS.md — 在 Brigade 代码库中工作

本指南面向为 Brigade 贡献代码的人类和 AI 编码代理。内容涵盖目录结构、命令、必须遵循的约定，以及常见改动的操作步骤。动手改代码前请先读一遍。

面向用户的文档见 [README.md](README.md)。发布流程见 [docs/RELEASING.md](docs/RELEASING.md)。

---

## Brigade 是什么

一个单操作者、多代理的 AI 团队运行时，用 TypeScript 编写。它以无头 WebSocket **网关**（持有状态的守护进程）形式运行，配轻量客户端（聊天 TUI、`connect`、渠道适配器）。它构建在 **pi** SDK 之上（`@earendil-works/pi-agent-core` + `@earendil-works/pi-ai` + `@earendil-works/pi-coding-agent` + `@earendil-works/pi-tui`，精确锁定 `0.79.9`），并在其上叠加产品层：记忆、技能、子代理、工具、组织层级、渠道、cron、扩展 SDK，以及双模式存储层。

- **技术栈：** TypeScript（严格模式）、ESM、Node **≥ 22.12**、**npm**（绝不用 pnpm）。
- **构建：** `tsc` → `dist/`。无打包器。
- **测试：** Node 内置的 `node:test`，通过 `tsx` 运行。
- **状态：** 全部位于 `~/.brigade/` 下（可用 `BRIGADE_STATE_DIR` 覆盖）。

---

## 命名与品牌约定（必须遵循）

- 产品名为 **Brigade**。在代码、注释、文案、CLI 帮助、错误消息、环境变量、守护进程标签和文档中，代理 SDK **只能**称为 `pi` / `@earendil-works/pi-coding-agent`。不得提及其他第三方 AI 代理项目。
- **绝不**提交密钥、真实个人数据（姓名、电话号码、邮箱）或本地绝对路径。使用占位符（`+1 555 010 0001`、`/path/to/project`）。
- 吉祥物是 🦁 the Pride。不得使用其他动物或符号。
- `templates/workspace/` 存放默认人格文件，**禁止改动**：行为修复应改代码，绝不改模板内容。
- 提交不得带 `Co-Authored-By` 尾注。

守护进程标签（不要重命名）：macOS launchd `com.brigade.gateway`，Linux systemd `brigade-gateway`，Windows 任务计划程序 `BrigadeGateway`。

---

## 命令

| 任务 | 命令 |
|---|---|
| 构建 | `npm run build`（`tsc -p tsconfig.build.json` → `dist/`） |
| 构建（监听） | `npm run build:watch` |
| 类型检查 | `npm run typecheck`（`tsc -p tsconfig.json --noEmit`，含测试） |
| 测试（全部） | `npm test`（`node scripts/run-tests.mjs`） |
| 测试（单文件） | `npm test -- src/agents/tools/registry.test.ts` |
| 记忆评估/基准 | `npm run bench` |
| 运行已构建二进制 | `npm run brigade -- <args>`（如 `npm run brigade -- agent -m "hi"`） |
| 开发（先构建再运行） | `npm run dev -- <args>` |
| 开发（不构建，tsx） | `npm run dev:tsx -- <args>` |
| 开发（自动重启） | `npm run watch` |
| 清理 | `npm run clean` |

开 PR 前，以下三项必须全部通过：`npm run typecheck`、`npm test`、`npm run build`。

**测试隔离：** `scripts/run-tests.mjs` 将 `BRIGADE_STATE_DIR` 固定到全新的临时目录，测试绝不会碰你真实的 `~/.brigade`。`src/` 下有 **281** 个 `.test.ts` 文件。

---

## 仓库结构

```
brigade/  (工作树；存储在 ~/.brigade)
├── brigade.mjs                 # bin shim：强制 Node ≥22.12，路由到 dist/entry.js
├── package.json                # npm；pi 0.79.9 精确锁定
├── tsconfig.json               # 类型检查配置（含测试）
├── tsconfig.build.json         # 构建配置（输出 dist/，排除测试/模板）
├── scripts/                    # run-brigade.mjs、run-tests.mjs、build-done.mjs、convex-*.mjs
├── skills/                     # 56 个内置技能目录
├── templates/workspace/        # 默认人格文件 — 禁止编辑
├── convex/                     # Convex schema + 函数（可选存储后端）
└── src/
    ├── entry.ts                # CLI 入口（快速路径 version/help，惰性分发）
    ├── extension-sdk.ts        # 公开插件 SDK（defineModule + 再导出）
    ├── cli/                    # 命令文件 + program/build-program.ts（命令注册表）
    ├── core/                   # 网关 server.ts、daemon/ 安装器、分发
    ├── agents/                 # 运行时（见下）— 代码主体
    │   ├── agent-loop.ts       # 每轮循环
    │   ├── session-wiring.ts   # 工具集装配 + 工具调用前守卫
    │   ├── tools/              # Brigade 原生工具 + registry.ts
    │   ├── memory/             # 事实存储、衰减、自动召回、整合
    │   ├── skills/             # 6 源发现、资格判定、管理
    │   ├── subagent-*.ts       # 生成、策略、中止级联、完成桥接
    │   ├── channels/           # 适配器契约 + inbound-pipeline + whatsapp/
    │   ├── routing/            # 入站 → (agentId, sessionKey)
    │   ├── org/                # org/Pride 层级 + A2A 策略
    │   └── extensions/         # 插件引擎 + 内置 modules/（网页搜索等）
    ├── tideline/               # 长期记忆引擎（混合召回、链接图）
    ├── system-prompt/          # assembler.ts + 各段（人格固定、组织锚点）
    ├── sessions/               # 会话存储、写锁、转录修复
    ├── storage/                # 双模式存储：local/（文件系统）+ convex/
    ├── config/                 # brigade.json schema、io、校验器
    ├── providers/              # 模型提供方目录 + 认证检测
    ├── auth/ · security/       # 认证配置、加密（libsodium seal）
    ├── cron/                   # 调度器 + 隔离代理运行执行器
    ├── tui/ · ui/              # 终端客户端 + 品牌框架
    └── workspace/              # 人格文件加载器（bootstrap.ts）
```

---

## 约定（不可协商）

### ESM 导入 — 始终带 `.js` 扩展名
```ts
import { makeAgentsListTool } from "./agents-list-tool.js";  // ✅
import { makeAgentsListTool } from "./agents-list-tool";     // ❌ 运行时出错
```
适用于所有相对导入。项目将 TS 编译为 ESM（NodeNext）。

### 严格 TypeScript
已开启 `strict`、`noUncheckedIndexedAccess` 和 `noImplicitOverride`。除不可避免的 schema/插件边界外，避免 `any`。测试必须通过 `npm run typecheck`。

### 所有权门控
特权工具采用两种姿态之一——优先第一种：
1. **按调用门控：** 工具对所有人注册，但根据 `opts.senderIsOwner` 分支，拒绝渠道对端的变更操作（如允许 `cron list`，不允许 `cron add`）。按操作粒度。
2. **一刀切 `ownerOnly: true`：** 注册时即拒绝非所有者。当*每个*操作都是特权时使用（如 `manage_provider`、`composio`）。`senderIsOwner` 默认为 `false`；只有显式所有者流程才设为 `true`。

### 记忆来源隔离（关键）
每次记忆读/写都贯穿一个 `MemoryRecordOrigin`（`owner` 与 `channel+conversationId+sessionKey`）。所有者事实对渠道对端永不可见；渠道事实仅在来源精确匹配时召回。去重仅限同来源。自动召回在注入**之前**按来源过滤。任何新的记忆路径都必须贯穿来源，否则隔离会静默失效。

### 配置与密钥
`brigade.json` 写入保留 5 份取证备份（`.bak` + `.bak.{1..4}`，权限 0600）。密钥使用 `${VAR_NAME}` 引用，读取时解析，写入时**还原**（不持久化解析后的值）。保持 `brigade secrets audit` 干净。

### 无按 cwd 的人格遍历器
Brigade **不**遍历项目规则文件。人格仅来自 `~/.brigade/workspace/`。

---

## 操作步骤

### 添加工具
1. 创建 `src/agents/tools/my-tool.ts`，导出 `makeMyTool()` 工厂，返回 `BrigadeTool`（TypeBox 参数、`jsonResult`）。
2. 在 `src/agents/tools/registry.ts`（`createBrigadeTools`）中注册。
3. 选择门控姿态（推荐按调用 `senderIsOwner`）。
4. 添加 `src/agents/tools/my-tool.test.ts`。添加工具时通常需更新三个枚举测试：**registry**、**session-wiring** 和 **owner-only** 测试。（断言精确工具数量的测试会中和 `COMPOSIO_API_KEY`——保持该模式。）

### 添加渠道
实现渠道适配器契约（config / gateway / outbound / security / status / message-action / secrets），在模块中通过 `b.channel(...)` 注册，惰性加载重依赖，实现带退避的重连 + 加密错误收窄 + JID 规范化，并通过去重 + 回复净化器 + 中止触发测试。WhatsApp（`src/agents/channels/whatsapp/`）是参考实现。

### 添加技能
在 `skills/`（内置）或 `~/.brigade/skills/`（托管，通过 `manage_skill`）下放一个目录。Frontmatter 需要 `name` + `description`（发现钩子），以及可选的资格元数据（`requires-bins`、`requires-env`、`requires-config`、OS）。不要引入按 cwd 的遍历器。

### 添加扩展提供方 / 模型提供方 / 网页搜索
创建 `src/agents/extensions/modules/<name>.ts`，从 `src/extension-sdk.ts` 注册到对应的 `b.*` 槽位。网页搜索模块带 `autoDetectOrder` 和 `isConfigured(cfg, env)` 谓词。

### 添加 CLI 子命令
在 `src/cli/commands/` 下添加文件，并在 `src/cli/program/build-program.ts` 中注册。如相关，更新 `brigade doctor` 检查。

---

## 存储模式（文件系统 vs. Convex）

Brigade 有双模式存储接缝（`src/storage/`）：`BrigadeStore` 接口含约 16 个类型化子存储，由 `local/`（文件系统，默认）和 `convex/`（可选）实现。模式在启动时从粘性哨兵解析，并在进程内冻结。用 `brigade store mode set <filesystem|convex>` 切换；用 `brigade store migrate` 复制数据。

改动存储时，保持两个后端对等，绝不要假设路径是文件系统形态。

### Convex 本地开发循环（完全自托管——无需云账号）

- `npm run convex:install` — 下载独立 `convex-local-backend` Rust 二进制 + 仪表盘到 `bin/`（已 gitignore）。许可证 FSL-1.1-Apache-2.0。
- `npm run convex:dev` — 安装（已缓存则跳过），然后启动后端（`http://127.0.0.1:3210`）、站点代理（`:3211`）和仪表盘（`:6791`）。同时写入 `.env.local`（已 gitignore），含自托管 URL + 管理密钥。
- `npm run convex:codegen` — 针对运行中的本地后端重新生成 `convex/_generated/`。

每机状态位于 `.convex-data/`（已 gitignore）：`identity.json`、`admin-key.txt`、SQLite 引擎文件、File Storage 和日志。`npx convex dev` 仅在 `.env.local` 中的自托管环境变量生效时才安全；否则它会尝试认领云项目——不要在该状态下运行。

---

## 架构不变量

- **认证文件**位于 `<agentDir>/agent/{auth-profiles,models}.json`，权限 0600。
- **会话**是 pi JSONL 转录，位于 `<agentDir>/sessions/<sessionId>.jsonl`，每会话一个文件，由 `sessions.json` 索引。有写锁和崩溃时转录修复。
- **绝不覆盖 `session.agent.streamFn`** — `createAgentSession` 安装了认证感知包装器；替换它会静默破坏所有模型调用。
- **人格固定会覆盖 pi 的技能注入** — Brigade 自行将技能渲染进装配的提示词（`applyPersonaOverrideToSession` 每进程锁存一次）。不要回退到 pi 自动注入。
- **`thinkingLevel` 必须推理感知** — 某些模型拒绝 `"off"`；从 `model.reasoning` 推导。
- **心跳**由工作区通过 `HEARTBEAT.md` 驱动；交付前从输出中剥离 `HEARTBEAT_OK` 令牌。
- **子代理深度**编码在会话键中；叶子代理（达到最大深度）完全失去生成工具，中止会级联到后代。

---

## 改动子系统时的主动检查

- `core/server.ts` / 网关 → 验证 thinking-persist、热重载、守卫扫描、模型设置测试通过。
- `agents/channels/` → 适配器契约得到遵守；WhatsApp 是参考；惰性加载纪律保持。
- `cron/` → `--cron` 配 IANA `--tz`；隔离运行留在 `~/.brigade/cron/runs/` 内。
- `agents/subagent-*` → 中止级联 + 完成排序 + 结算唤醒通过。
- `agents/tools/registry.ts` → pi 内置工具完好（`read`/`write`/`edit`/`bash`/`grep`/`find`/`ls`）；门控姿态记录在工具的首行注释中。
- `agents/memory/` → 来源贯穿保持；去重仅同来源；自动召回在注入前过滤。
- `system-prompt/` → 装配器保留规范段落和人格固定。

---

## 提交与发布策略

- 使用 [Conventional Commits](https://www.conventionalcommits.org/)。`feat` / `fix` / `perf` / `deps` / `revert` 触发发布；`docs` / `refactor` / `test` / `ci` / `chore` 不触发。
- 提交消息解释*为什么*，而非*做了什么*。不要 `Co-Authored-By: Claude` 尾注。
- 发布通过 release-please 自动化 → npm publish（`@spinabot/brigade`）。见 [docs/RELEASING.md](docs/RELEASING.md)。