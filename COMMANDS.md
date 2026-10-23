# 🦁 Brigade — 命令参考

按界面分组的全部可端到端运行的命令。**唯一事实来源：[`src/cli/program/build-program.ts`](src/cli/program/build-program.ts)** — 该文件中的命令变更时，本文档也应同步更新（见文末[保持本文档同步](#保持本文档同步)）。

> 约定
> - `<尖括号>` 内为必填参数；`[方括号]` 内为可选参数。
> - 所有只读命令均支持 `--json`，输出机器可读结果。
> - 开发运行器为 `node F:/Brigade/brigade.mjs <…>`（或 `npm run start -- <…>`）。执行 `npm install -g .` 后可直接输入 `brigade …`。

---

## 🚀 快速开始

```bash
# 1. 一次性配置 provider
brigade onboard

# 2. 日常使用 — 进程内 TUI（自动启动 gateway）
brigade

# 或 — 守护进程 + 瘦客户端（长会话推荐）
brigade gateway          # 终端 A
brigade connect          # 终端 B
```

---

## 🎛️ 顶层命令

| 命令 | 说明 |
|---|---|
| `brigade` *（默认）* / `brigade tui` | 启动聊天 TUI（必要时自动启动 gateway） |
| `brigade onboard` | 选择 provider + 模型 — 交互式向导 |
| `brigade agent <prompt>` | 通过 agent 流水线执行单轮对话（一次性、非交互） |
| `brigade connect` | 连接运行中 gateway 的瘦 TUI 客户端 |
| `brigade status` | Brigade 配置、会话、gateway 状态快照 |
| `brigade doctor` | 完整健康检查（配置 / provider / 日志接收器 / 系统提示词组装 / 可选 gateway 探测） |
| `brigade logs [--follow]` | 跟踪今日 gateway 日志 |

### 通用标志

| 标志 | 适用命令 | 含义 |
|---|---|---|
| `--host <host>` | gateway/connect/status/doctor | Gateway 主机（默认 `127.0.0.1`） |
| `--port <port>` | gateway/connect/status/doctor | Gateway 端口（默认 `7777`） |
| `--json` | 几乎所有命令 | 机器可读输出 |
| `--no-env-detect` | onboard, tui | 忽略 shell 环境中的 API key |

---

## 🛰️ Gateway

```bash
brigade gateway run                  # 前台运行；裸 `brigade gateway` 的别名
brigade gateway status               # 守护进程是否运行？在哪个端口？
brigade gateway stop [--timeout <ms>]
brigade gateway install              # 注册为操作系统服务（launchd / systemd / Task Scheduler）
brigade gateway uninstall
brigade gateway restart
```

`run` / 裸 `brigade gateway` 接受的 gateway 级标志：

| 标志 | 作用 |
|---|---|
| `-p, --port <n>` | 绑定的 TCP 端口 |
| `-h, --host <addr>` | 绑定的网络接口 |
| `-V, --verbose` | 日志级别提升至 `debug` |
| `-q, --quiet` | 关闭控制台输出流 |
| `--log-level <lvl>` | `trace`/`debug`/`info`/`warn`/`error`/`fatal` 之一 |

---

## ⚙️ 配置

`brigade.json` 位于 `~/.brigade/brigade.json`（或 `%USERPROFILE%\.brigade\brigade.json`）。可直接编辑，或使用以下命令：

| 命令 | 说明 |
|---|---|
| `brigade config list` | 打印完整配置（敏感信息已脱敏） |
| `brigade config get <path>` | 获取点路径处的单个值（如 `agents.defaults.provider`） |
| `brigade config set <path> <value>` | 设置值（JSON 或字符串）— 支持 `--strict-json` + `--dry-run` |
| `brigade config unset <path>` | 删除键 |
| `brigade config file` | 打印配置文件的绝对路径 |
| `brigade config schema` | 以 JSON 打印 TypeBox Schema |
| `brigade config validate` | 按 Schema 校验配置；有问题时以非零码退出 |

---

## 📡 渠道

WhatsApp、Slack、Telegram、Discord 等 — 所有通过 Brigade 渠道适配器接入的渠道。

```bash
brigade channels list                          # 哪些渠道已配置 / 已关联 / 已启动
brigade channels status [--channel <id>]
brigade channels link [--channel <id>] [--force] [--timeout <ms>]
brigade channels unlink [--channel <id>] [-y]
brigade channels enable [--channel <id>]
brigade channels disable [--channel <id>]
```

### 允许列表（谁可以向 agent 发私信）

```bash
brigade channels allow list [--channel <id>]
brigade channels allow add <senderId> [--channel <id>]      # 如 +15551234567
brigade channels allow remove <senderId> [--channel <id>]
```

### 配对（新设备连接的运维方审批）

```bash
brigade pairing list                      # 显示待处理的配对码
brigade pairing approve <code>
brigade pairing revoke <code>
```

---

## 🧑‍🤝‍🧑 Agents — 隔离人格

每个 agent 拥有独立工作区（人格文件）、认证配置、exec 允许列表、会话，以及可选的渠道/账户路由绑定。`brigade.json` 将其存储在 `cfg.agents.<id>`（键值映射）和 `cfg.bindings.entries[]`（路由规则）下。裸 `brigade agents` 默认执行 `list`。

```bash
brigade agents                                                # 列出所有 agent（默认）
brigade agents list [--json] [--bindings]
brigade agents bindings [--agent <id>] [--json]
brigade agents bind --agent <id> --bind <spec> [--bind <spec> …] [--json]
brigade agents unbind --agent <id> (--bind <spec> [--bind <spec> …] | --all) [--json]
brigade agents add <name> [--workspace <dir>]
                  [--model <id>] [--provider <id>] [--agent-dir <dir>]
                  [--bind <spec> [--bind <spec> …]]
                  [--non-interactive] [--json]
brigade agents set-identity --agent <id>
                  [--workspace <dir>] [--identity-file <path>] [--from-identity]
                  [--name <…>] [--theme <…>] [--emoji <…>] [--avatar <…>]
                  [--json]
brigade agents delete <id> --force [--json]
```

**绑定规格格式**

绑定规格为裸渠道 id（`whatsapp`、`telegram`、`discord`），或 `<channel>:<accountId>` 以将绑定限定到单个账户（`whatsapp:+15551234567`）。绑定在 agent 间先到先得 — 已被其他 agent 占用的槽位会报告为冲突（退出码 1），而非静默改路由。

**保留 id**

`main`（默认 agent）、`none`、`null`、`undefined`、`default`、`all`、`any` 为保留字，不能用作 agent id。`main` 还不能被 `delete`。

**工作区默认值**

`agents add <name>` 不带 `--workspace` 时自动默认为 `~/.brigade/agents/<name>/workspace/`。仅在需要自定义位置时传入 `--workspace <dir>`。

**交互式对齐**

`add`、`delete`、`set-identity` 先提供非交互模式。交互式向导（工作区选择器、身份提示、删除确认）为后续工作；在其落地前，这些命令会以清晰消息明确报错，而非阻塞在 TTY 提示上。

---

## 🗂️ 会话

每个 agent 的对话记录位于 `~/.brigade/agents/<id>/sessions/`。

```bash
brigade sessions list [--agent <id>]                 # 最新在前
brigade sessions cleanup --older-than <30d|12h>      # 支持 --dry-run
```

---

## 🧠 技能

从内置目录 + 工作区目录自动发现的提示词常驻技能。

```bash
brigade skills list                  # 组装器可见的全部技能
brigade skills info <name>           # 完整 SKILL.md 正文
```

---

## 🔐 Exec — bash 审批门

Brigade 的 bash 工具受门控。运维方构建安全命令允许列表。

```bash
brigade exec list                                    # 显示允许列表（命令 + 模式）
brigade exec allow <command…>                        # 批准精确命令
brigade exec allow-pattern <regex>                   # 批准正则模式
brigade exec remove <command-or-pattern>             # 从允许列表移除
brigade exec deny-test <command…>                    # 门控会如何分类此命令？
brigade exec file                                    # exec-approvals.json 的绝对路径
```

均接受 `--agent <id>` 以限定到非默认 agent（默认 `main`）。

**示例**
```bash
brigade exec allow ls -la
brigade exec allow-pattern '^git (status|diff|log)( |$)'
brigade exec deny-test 'rm -rf /'
```

---

## ⏰ Cron — 定时任务

每个 agent 的周期性 / 一次性任务。服务运行在 gateway 中；此 CLI 直接操作 `~/.brigade/cron.json`。

```bash
brigade cron status                       # 服务健康状态
brigade cron list [--all] [--query <text>] [--limit <n>]
brigade cron add  --name <name> (--at <iso|ms> | --every <5m|1h> | --cron <expr>)
                  (--message <prompt> | --system-event <text>)
                  [--tz <iana>] [--target main|isolated|session:<id>]
                  [--model <id>] [--thinking off|low|medium|high]
                  [--timeout-seconds <n>] [--tools <csv>] [--light-context]
                  [--deliver | --no-deliver]
                  [--channel <id>] [--to <addr>] [--account <id>]
                  [--best-effort-deliver] [--disabled]
brigade cron edit <jobId>   [--name <…>] [--description <…>] [--enable | --disable]
brigade cron rm <jobId>     # 别名：remove / delete
brigade cron enable <jobId>
brigade cron disable <jobId>
brigade cron run <jobId>    [--due]    # 立即触发（强制）或仅在超过下次触发时间时触发（due）
brigade cron runs <jobId>   [--limit <n>]
```

**调度格式**

| 类型 | 示例 | 用途 |
|---|---|---|
| `--at` | `--at 2026-06-04T02:00:00+05:30` 或 `--at 1780510620000` | 一次性触发 |
| `--every` | `--every 5m` / `--every 1h` / `--every 30s` | 固定间隔周期 |
| `--cron <expr> --tz <iana>` | `--cron "0 9 * * *" --tz Asia/Kolkata` | 日历对齐周期 |

> ⚠️ **`--cron` 务必搭配 `--tz`**，使用完整 IANA 时区（`Asia/Kolkata`、`America/Los_Angeles`、`Europe/London`）。`IST` / `PT` 等缩写不是有效 IANA 时区。

**载荷格式**

| 标志 | 搭配 | 作用 |
|---|---|---|
| `--message <prompt>` | 默认 `target=isolated` | 触发时运行一次全新 agent 轮次执行该提示词 |
| `--system-event <text>` | 默认 `target=main` | 文本注入运维方主会话收件箱 |

---

## 💾 备份

```bash
brigade backup create [--out <path>]            # brigade.json + 工作区 + 技能 + 会话的归档
brigade backup verify <archive>                 # 校验和 + 结构检查
brigade backup restore <archive>                # 恢复到全新的 ~/.brigade/
```

---

## 🔍 诊断

```bash
brigade doctor [--strict]                       # 完整健康检查；--strict 在警告时以非零码退出
brigade secrets audit                           # 扫描配置 / 磁盘中的意外密钥泄露
brigade logs [--follow]                         # 跟踪今日 gateway 日志
```

---

## ⌨️ TUI 斜杠命令

在 `brigade` / `brigade connect` / `brigade chat` 内可输入：

| 斜杠 | 作用 |
|---|---|
| `/help` | 完整斜杠命令列表（实时） |
| `/usage` | 本会话的 token + 费用汇总 |
| `/model [provider id]` | 会话中切换模型 |
| `/thinking [off\|low\|medium\|high]` | 调整推理预算 |
| `/compact` | 立即强制压缩对话记录 |
| `/reasoning [on\|off]` | 切换是否显示思考块 |
| `/abort` | 中止进行中的轮次（或 Ctrl+C） |
| `/agent [id]` | 切换连接绑定的 agent |
| `/session [key]` | 切换连接绑定的会话 |
| `/agents` | 列出已配置的 agent |
| `/sessions [--all]` | 列出绑定 agent 的会话（或全部） |
| `/mute <id\|key>` | 服务端取消订阅某 agent 或会话 |
| `/provider` *（仅 chat，不含 connect）* | 内联重新运行 provider 引导 |

> `connect` 模式有意省略 `/provider` — 引导会写入 gateway 机器的文件系统，瘦客户端无法访问。

---

## 🛠️ npm 脚本（开发者模式）

在 `F:/Brigade` 检出目录中：

| 脚本 | 作用 |
|---|---|
| `npm run dev` | 对当前源码运行 bin shim — 若 `dist/` 过期则自动重建 |
| `npm run dev -- onboard`（等） | 同上，但传入子命令参数 |
| `npm run start` | 同 `dev` |
| `npm run tui` | 通过开发运行器直接启动 TUI |
| `npm run gateway` | 通过开发运行器直接启动 gateway |
| `npm run onboard` | 通过开发运行器运行引导 |
| `npm run agent -- <prompt>` | 通过开发运行器执行一次性 agent 轮次 |
| `npm run build` | `tsc -p tsconfig.build.json` → `dist/` |
| `npm run typecheck` | `tsc --noEmit`（不输出） |
| `npm test` | `tsx --test src/**/*.test.ts` — 运行完整测试套件 |
| `npm run clean` | 删除 `dist/` |
| `npm run watch` | `tsx --watch src/entry.ts` 用于实时开发 |

---

## 📁 文件位置

| 路径 | 内容 |
|---|---|
| `~/.brigade/brigade.json` | 主配置（原子写入 + `.bak.{1..4}` 轮转） |
| `~/.brigade/auth.json` | Provider 认证（权限 0600） |
| `~/.brigade/models.json` | 自定义 provider 目录 |
| `~/.brigade/cron.json` | Cron 任务存储（锁序列化） |
| `~/.brigade/cron/runs/<jobId>.jsonl` | 每个任务的运行历史 |
| `~/.brigade/agents/<id>/workspace/` | 人格文件（`SOUL.md` / `IDENTITY.md` / `AGENTS.md` / `TOOLS.md` / `USER.md` / `BOOTSTRAP.md` / `HEARTBEAT.md`） |
| `~/.brigade/agents/<id>/sessions/` | JSONL 对话记录 |
| `~/.brigade/agents/<id>/agent/auth-profiles.json` | 每个 agent 的认证配置 |
| `~/.brigade/agents/<id>/workspace/memory/facts.jsonl` | 每个 agent 的持久事实 |
| `~/.brigade/agents/<id>/exec-approvals.json` | 每个 agent 的 bash 允许列表 |
| `~/.brigade/logs/<date>.jsonl` | 结构化 gateway 日志 |
| `~/.brigade/channels/<channel>/<accountId>/` | 每个账户的渠道状态（认证、允许来源、配对） |
| `~/.brigade/workspace/memory/` *（旧版）* | 顶层工作区记忆（旧安装） |

---

## 保持本文档同步

`COMMANDS.md` 为**手工维护**，但锚定于单一事实来源。修改 CLI 时：

1. 规范命令注册位于 [`src/cli/program/build-program.ts`](src/cli/program/build-program.ts)。运行 `grep -nE '\.command\(|\.description\