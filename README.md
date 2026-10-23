

> **[中文化] brigade**
>
> 此项目是 [spinabot/brigade](https://github.com/spinabot/brigade) 的中文翻译版本。
> - 原项目 Stars: 0
> - 主语言: 
> - 许可证: 
> - 翻译日期: 2026-10-23
> - 原始 README: [README_en.md](README_en.md)
>
> 说明：下方 README 正文为**原项目文档的中文翻译**，力求忠于原文；本页顶部的「中文快速上手」为**面向中文用户的补充导读**（非原文内容），目的是让你不用读完整篇 README 就能先把项目跑起来。
>
> 如有翻译不准确之处，欢迎提 Issue 或 PR。

---


## 🚀 中文快速上手（补充导读，非原文）

### 这是什么 / 解决什么问题

Brigade 是一个自托管的个人 AI 智能体（agent）系统：一组 AI agent 按「组织架构」协作，共享一套名为 Tideline 的长期记忆，可以互相委派任务、在任务中途切换模型，并接入 1000+ 应用。它解决的是「把 AI 助手做成一个可自己掌控的团队」这件事——不需要注册账号、没有中间 SaaS、无遥测，API Key 和数据都留在你自己的机器上。它同时提供终端聊天界面，以及 WhatsApp、Telegram、Slack、Discord、iMessage 等消息渠道入口。

### 快速上手（最短路径）

1. 安装（macOS / Linux，脚本会在需要时安装 Node，无需 sudo）：
   ```bash
   curl -fsSL https://brigade.spinabot.com/install.sh | sh
   ```
2. 启动：
   ```bash
   brigade
   ```
3. 启动后会进入一个聊天 TUI，直接开始对话即可。
4. 如需把 agent 暴露到公网（README 中的 B³ 基准测试相关功能），可用：
   ```bash
   brigade expose
   ```
   停止暴露：
   ```bash
   brigade expose stop
   ```
5. 其余 CLI 子命令、聊天内命令、配置项等，原文未在提供的材料中展开，请参考 README 正文的「CLI reference」「In-chat commands」「Configuration & storage」等章节。

### 环境要求

- **操作系统**：README 给出的安装命令标注为 macOS / Linux；Windows 是否支持，原文未说明。
- **硬件**：原文称同一套 crew 可运行在 Raspberry Pi 或服务器上，未给出具体最低配置。
- **运行时**：Node.js（安装脚本会在需要时自动安装 Node；package.json 中声明了 node engine 字段，但具体版本号在提供的材料中未显示）。
- **依赖**：默认是小型文件系统安装；可选切换到自托管的 Convex 数据库（原文未说明切换所需的具体依赖与版本）。
- **模型**：支持 Claude、GPT、Gemini、Llama 或本地 Ollama；也可用 Claude / ChatGPT / Copilot 订阅登录，或复用已登录的 Claude Code / Codex CLI 会话。
- **网络**：安装脚本从 `brigade.spinabot.com` 下载，国内网络环境如遇下载缓慢，可能需要自备加速方式（原文未提供镜像源）。

### 常见坑与注意事项

- **需要模型访问权限**：要么提供 API Key，要么用 Claude / ChatGPT / Copilot 订阅登录，或复用已有的 Claude Code / Codex CLI 登录；原文强调 Key 只留在本机、不经中间人。
- **`brigade expose` 会把 agent 暴露到公网**：原文说明默认走 Cloudflare，也可自带 `bore` / `frp` / `sish` 中继，并带一个不可见的密钥做访问控制（未授权访问返回 401）。使用前需清楚这是对外暴露服务。
- **数据都在 `~/.brigade/` 下**：原文称该目录完全归你所有，迁移或备份时注意这个路径。
- **存储模式可选**：默认文件系统，切到自托管 Convex 需要额外部署，原文未说明具体步骤与资源要求。

---



---

# 🦁 Brigade — 你的个人智能，企业级构建

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/spinabot/brigade/main/assets/brigade-banner-on-black.gif" />
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/spinabot/brigade/main/assets/brigade-banner.gif" />
    <img src="https://raw.githubusercontent.com/spinabot/brigade/main/assets/brigade-banner.gif" alt="Brigade — 你的个人智能，企业级构建" width="900" />
  </picture>
</p>

<p align="center">
  <strong>为企业规模而生。向所有人开放。</strong> &middot; <em>一个生态，而非一个应用。</em>
</p>

<p align="center">
  🔐 <strong>无需管理令牌。没有麻烦。</strong> 无需复制粘贴密钥，无遥测 —
  <strong>我们替你处理，而非你亲自处理</strong>。你的 API 密钥留在自己的机器上，绝不发送给中间人。
</p>

<p align="center">
  🔑 <strong>没有 API 密钥？没关系。</strong> 用你的 <strong>Claude</strong>、<strong>ChatGPT</strong> 或
  <strong>Copilot</strong> 订阅登录 — 或者，如果你已登录 <strong>Claude Code</strong> 或
  <strong>Codex</strong> CLI，直接复用该登录（无需浏览器，无需重新认证）。
</p>

<p align="center">
  🩸💥 <strong>B³ — Brigade 血腥基准测试</strong> 💥🩸<br/>
  <sub><strong>v1.9.0 新增</strong> · 一条命令把你的团队扔进狼群（充满爱意地）</sub>
</p>

<p align="center">
  <strong><em>三个 B。一条血腥命令。你的团队 — 放归开放互联网。</em></strong>
</p>

<p align="center">
  <strong>Brigade</strong> 是你的团队。<strong>Bloody</strong> 是你对它做的事 — 无安全网地把它抛向开放世界，沿一条炽热隧道冲向荒野。<strong>Benchmark</strong> 是点睛之笔：唯一真正重要的测试不在实验室，而是<em>陌生人在凌晨 3 点戳你的智能体</em>。扛过这个，你就上线了。
</p>

<p align="center">
  底层其实很温和 — 边缘用 HTTPS，无账号，无配置。默认走 Cloudflare，也可自带中继（<code>bore</code> · <code>frp</code> · <code>sish</code>，均为开源）。一个密钥<strong>隐形</strong>随行：你永远看不到它，闯入者只会在 <code>401</code> 上流血而亡。不喜欢这个名字？它也响应正经的 <code>brigade expose</code>。🩸
</p>

<p align="center">
  <code>brigade bloody&nbsp;benchmark</code> &nbsp;·&nbsp; <code>brigade expose</code> &nbsp;·&nbsp; <code>brigade expose stop</code> &nbsp;🩸
</p>

<p align="center">
  <a href="https://trendshift.io/repositories/118354?utm_source=trendshift-badge&amp;utm_medium=badge&amp;utm_campaign=badge-trendshift-118354" target="_blank" rel="noopener noreferrer"><img src="https://trendshift.io/api/badge/trendshift/repositories/118354/daily?language=TypeScript" alt="spinabot/brigade 在 Trendshift 上" width="250" height="55" /></a>
</p>

<p align="center">
  <a href="https://github.com/spinabot/brigade/actions/workflows/ci.yml"><img src="https://img.shields.io/github/actions/workflow/status/spinabot/brigade/ci.yml?branch=main&style=for-the-badge&logo=githubactions&logoColor=white&label=CI" alt="CI 状态"></a>
  <a href="https://www.npmjs.com/package/@spinabot/brigade"><img src="https://img.shields.io/npm/v/@spinabot/brigade?style=for-the-badge&logo=npm&logoColor=white&color=CB3837" alt="npm 版本"></a>
  <a href="https://www.npmjs.com/package/@spinabot/brigade"><img src="https://badgen.net/npm/dt/@spinabot/brigade?icon=npm&color=CB3837" alt="npm 总下载量"></a>
  <a href="https://nodejs.org"><img src="https://img.shields.io/node/v/@spinabot/brigade?style=for-the-badge&logo=nodedotjs&logoColor=white&color=5FA04E&label=node" alt="node 引擎"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge" alt="MIT 许可证"></a>
  <a href="https://brigade.spinabot.com"><img src="https://img.shields.io/badge/website-brigade.spinabot.com-E8B021?style=for-the-badge&logo=googlechrome&logoColor=white" alt="网站"></a>
</p>

**Brigade 是你的个人智能，构建于企业级技术之上**：一支 AI 智能体团队，运行在真实组织架构图上，共享同一个长期记忆 Tideline，因此一个智能体学到的，其余都能使用。它们互相委派任务，可在任务中途切换模型而不丢失上下文，并在你已在使用的 1,000+ 应用中执行操作。这是一个你自行托管的生态，具备企业级技术的管控与数据主权：无需创建账号，中间没有 SaaS。

同一支团队可运行在树莓派或服务器上。默认是小巧的文件系统安装；需要时可切换到自托管 Convex 数据库。可接入任意模型：Claude、GPT、Gemini、Llama 或本地 Ollama。特权操作等待你批准，你的密钥和数据永不离开你的机器。无遥测。

这是一个生态，而非一个应用：一支团队，你可从终端、WhatsApp、Telegram、Slack、Discord、iMessage 和 BlueBubbles 访问，也可从手表、Meta 智能眼镜和 Meta Quest 访问。

```bash
# macOS / Linux — 按需安装 Node，然后安装 Brigade（无需 sudo）
curl -fsSL https://brigade.spinabot.com/install.sh | sh
brigade
```

启动后你会得到一个快速的聊天 TUI。继续深入，你会得到拥有独立工作区和凭据的隔离智能体、持久记忆、技能系统、cron 调度器、子智能体扇出、组织层级、WhatsApp 和 Telegram 等消息渠道、1,000+ 应用连接器、MCP 记忆服务器，以及可选的自托管 Convex 后端 — 全部位于你完全拥有的 `~/.brigade/` 目录下。

> 一个所有者，一整支团队 — 众多智能体通过真实组织架构图协调，全部运行在**你**掌控的硬件和存储上。

---

## 目录

- [为什么选择 Brigade](#why-brigade)
- [安装](#install)
- [快速开始](#quick-start)
- [工作原理](#how-it-works)
- [功能](#features)
- [CLI 参考](#cli-reference)
- [聊天内命令](#in-chat-commands)
- [内置工具](#built-in-tools)
- [团队模式](#team-mode)
- [多智能体隔离](#multi-agent-isolation)
- [提供商与网络搜索](#providers--web-search)
- [配置与存储](#configuration--storage)
- [存储模式：文件系统 vs. Convex](#storage-modes-filesystem-vs-convex)
- [MCP 记忆服务器](#mcp-memory-server)
- [Tideline（长期记忆）](#tideline-long-term-memory)
- [扩展 Brigade](#extending-brigade)
- [隐私](#privacy)
- [从源码开发](#develop-from-source)
- [贡献](#contributing)
- [许可证](#license)
- [🦁 Brigadiers](#-brigadiers)

---

## 为什么选择 Brigade

| | |
|---|---|
| 🖥️ **终端优先** | 默认无闪烁的聊天 TUI。无浏览器，无 Electron。 |
| 🧠 **持久记忆** | 事实跨会话持久化，具备来源作用域、衰减和混合（关键词 + 向量）召回 — **Tideline** 引擎。 |
| 👥 **真实团队** | 生成拥有独立人格、凭据和记忆的隔离智能体；将它们接入**组织架构图**，管控谁能与谁对话。 |
| 🔌 **接入任意模型** | Anthropic、OpenAI、Gemini、OpenRouter、OrcaRouter、Token Market、Groq、Cerebras、xAI、DeepSeek、Mistral、本地 Ollama，或任意 OpenAI 兼容端点。对话中途切换。 |
| 📅 **常驻运行** | 作为无头 WebSocket 网关运行，带崩溃监控、cron 任务和操作系统服务安装。 |
| 💬 **渠道** | 目前可从 WhatsApp、Telegram、Slack、Discord、iMessage 和 BlueBubbles 与团队对话；适配器契约支持更多。 |
| 🔗 **1,000+ 连接器** | Gmail、Slack、GitHub、Notion、Calendar、Linear… 通过内置 Composio 工具。 |
| 📄 **读写文档** | 发送 PDF、Office 文档、图片、音频或视频，你的团队能*理解*它 — 然后让它**创建或编辑** Word、Excel、PowerPoint 和 PDF 文件并交还给你。 |
| 🧩 **MCP** | 将长期记忆暴露给任意 MCP 客户端（`brigade mcp`），或接入 MCP 服务器。 |
| 🗄️ **你的存储** | 默认文件系统模式，或可选的**完全自托管 Convex** 后端，带静态加密。 |
| 🔐 **属于你** | 一切位于 `~/.brigade/` 下。密钥以 `0600` 模式本地存储。`rm -rf ~/.brigade` 即可彻底清除。 |

---

## 安装

**最快 — 替你处理 Node**（若未安装则安装 Node 22.12+，然后安装 Brigade）：

```bash
# macOS / Linux
curl -fsSL https://brigade.spinabot.com/install.sh | sh
# …或直接从 GitHub：
curl -fsSL https://raw.githubusercontent.com/spinabot/brigade/main/packaging/install/install.sh | sh
```

```powershell
# Windows (PowerShell)
irm https://brigade.spinabot.com/install.ps1 | iex
# …或直接从 GitHub：
irm https://raw.githubusercontent.com/spinabot/brigade/main/packaging/install/install.ps1 | iex
```

**已在 Node 22.12+ 上？** 直接从 npm 安装：

```bash
npm i -g @spinabot/brigade
```

> **遇到 `EACCES` / 权限拒绝？** 你的 Node 全局文件夹需要管理员权限。**不要用 `sudo`** — 要么使用上面的 `curl … | sh` 安装器（它设置一个私有的、用户拥有的运行时，无需 root），要么将 npm 指向用户可写的前缀：`npm config set prefix ~/.npm-global`，然后将 `~/.npm-global/bin` 加入你的 `PATH`。

然后启动引导：

```bash
brigade
```

---

## 快速开始

```bash
# 1. 设置 — 选择提供商，粘贴 API 密钥（或扫描本地 Ollama），选择模型
brigade onboard

# 2. 启动常驻网关 — 你的团队、渠道和 cron 任务都在这里运行
brigade gateway run

# 3. 与团队聊天
brigade tui
```

引导向导带你走完**五步**：**(0) 存储模式**（文件系统或自托管 Convex）、**(1) 选择提供商**（Anthropic、OpenAI、Gemini、OpenRouter、OrcaRouter、Token Market、Ollama 等）、**(2) 连接**（粘贴 API 密钥、**用 Claude/ChatGPT/Copilot 订阅登录**、复用 Claude Code 或 Codex CLI 登录，或连接本地 Ollama）、**(3) 选择默认模型**、**(4) 网络搜索**（选择搜索后端；无密钥选项开箱即用）。后续启动从上次中断处继续。

`brigade tui`（或直接 `brigade`）若网关未运行会自动启动，因此快速本地会话可跳过第 2 步。要让网关跨重启存活，运行 `brigade gateway install`（安装 launchd / systemd / Task Scheduler 服务）。

其他便捷入口：

```bash
brigade agent -m "summarize ~/today.md"   # 单次对话，无 TUI
brigade connect                            # 将客户端连接到运行中的网关
brigade doctor                             # 健康检查你的安装
```

---

## 工作原理

Brigade 是一个二进制文件，可作为**网关**（持有全部状态并运行智能体的长生命周期进程）或作为连接到它的**瘦客户端**。

```
            ┌─────────────────────────── ~/.brigade/ ───────────────────────────┐
            │  config · per-agent workspaces · sessions (JSONL) · memory · cron  │
            └───────────────────────────────────────────────────────────────────┘
                                          ▲
                                          │ reads/writes (filesystem OR Convex)
                        ┌─────────────────┴─────────────────┐
   brigade tui ──ws──▶  │            GATEWAY  :7777          │  ◀──ws── brigade connect
   brigade chat         │  per-turn agent loop · routing    │
   WhatsApp·Telegram ──▶│  tools · sub-agents · supervisor  │◀──────── cron jobs
   MCP client ─stdio──▶ │  (brigade mcp → memory server)    │
                        └───────────────────────────────────┘
```

- **客户端是瘦的。** TUI（`brigade`/`brigade tui`）、`brigade connect` 和渠道适配器都是网关的 WebSocket 客户端。状态只存在于网关；客户端镜像它。断开、离开、稍后重连 — 你的智能体继续运行。
- **一次一轮。** 每条消息经过弹性智能体循环：解析会话、组装系统提示（人格 + 技能 + 组织上下文）、流式调用模型、通过审批和所有权守卫执行工具调用、持久化记录、回复。
- **来自任何地方。** WhatsApp 消息和 TUI 按键流经同一路由管道，解析为 `(agent, session)` 对，因此一个渠道上的对等方绝不会渗入另一个的上下文。
- **保持运行。** 心跳文件加进程外监控器在事件循环卡住时重启网关；各操作系统服务安装器在登录时启动它。

---

## 功能

### 🧠 记忆（Tideline）
文件支撑的事实（`facts.jsonl`），标记**来源**（所有者 vs. 渠道对等方），因此对等方的事实绝不会泄漏到你的上下文。事实按三个层级**衰减**，写入时去重，位于**溯源写入门**之后，阻止不可信来源覆盖你的身份/偏好。**Tideline** 引擎增加**混合召回**（BM25 关键词 + 无模型向量通道）、类型化链接图和夜间反思/整合过程。自动召回仅在每轮前注入来源匹配的事实。工具：`recall_memory`、`read_memory`、`write_memory`、`manage_memory`。→ **[docs/tideline.md](docs/tideline.md)**

### 🪪 技能
将含 `SKILL.md` 的文件夹放入工作区，Brigade 下一轮即发现它。技能是**提示常驻**的（注入名称 + 描述；模型按需读取正文）。六个发现根按优先级顺序合并，带操作系统、所需二进制、环境变量和配置的资格过滤。内置 56 个技能。用 `manage_skill` 管理。

### 👥 子智能体
将工作委派给子智能体：`spawn_agent`（单个）和 `spawn_agents`（并行扇出）。深度上限（默认 3）和子数量上限（默认 5），带**中止级联**和完成桥接，在子智能体完成时唤醒父级。

### 🏢 组织 / Pride 层级
将团队定义为真实组织架构图（`cfg.org`）：成员、部门、汇报线。**A2A（智能体对智能体）策略**从图中派生，管控哪些智能体可向哪些发消息。用 `brigade org show` / `explain` / `doctor` 检查，或用 `org` 工具渲染 🦁 Pride 图。

### 💬 渠道
类型化渠道适配器契约，带共享入站管道（访问控制、去重、中止触发、审批路由）。**WhatsApp**（Baileys，QR/码配对，多账号，重连退避）、**Telegram**（Bot API 令牌，内联按钮审批，网关启动时引导所有者）、**Slack**（Socket Mode，bot + app 令牌，频道/线程聊天）、**Discord**（bot 令牌，DM/服务器/线程聊天）、**iMessage**（通过 `imsg` CLI，仅纯文本）和 **BlueBubbles**（macOS 服务器，REST 出 + webhook 入，反应/编辑/撤回）现已发布；契约支持更多。完整演练见 [Telegram 设置](#telegram-setup-end-to-end)。

### ⏰ Cron
调度重复或一次性工作：cron 表达式（带 IANA 时区和错峰）、固定间隔或一次性时间戳