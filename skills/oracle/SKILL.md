---
name: oracle
description: 使用 oracle CLI 的最佳实践（提示词 + 文件打包、引擎、会话和文件附加模式）。
homepage: https://askoracle.dev
metadata:
  {
    "brigade":
      {
        "emoji": "🧿",
        "requires": { "bins": ["oracle"] },
        "install":
          [
            {
              "id": "node",
              "kind": "node",
              "package": "@steipete/oracle",
              "bins": ["oracle"],
              "label": "安装 oracle（node）",
            },
          ],
      },
  }
---

# oracle — 最佳用法

Oracle 将你的提示词和选定文件打包成一个“一次性”请求，让另一个模型基于真实仓库上下文作答（API 或浏览器自动化）。输出仅供参考：务必对照代码和测试验证。

## 主要用例（浏览器，GPT‑5.2 Pro）

默认工作流：在 ChatGPT 中使用 `--engine browser` 搭配 GPT‑5.2 Pro。这是常见的“长时间思考”路径：10 分钟到 1 小时属正常；会保存会话，可重新接入。

推荐默认值：

- 引擎：浏览器（`--engine browser`）
- 模型：GPT‑5.2 Pro（`--model gpt-5.2-pro` 或 `--model "5.2 Pro"`）

## 黄金路径

1. 选取精简的文件集（包含关键信息的最少文件）。
2. 预览载荷和 token 消耗（`--dry-run` + `--files-report`）。
3. 常规 GPT‑5.2 Pro 工作流用浏览器模式；仅在明确需要时用 API。
4. 若运行脱离或超时：重新接入已保存的会话（不要重跑）。

## 命令（推荐）

- 帮助：
  - `oracle --help`
  - 若未安装二进制：`npx -y @steipete/oracle --help`（此处避免 `pnpx`；sqlite 绑定问题）。

- 预览（不消耗 token）：
  - `oracle --dry-run summary -p "<task>" --file "src/**" --file "!**/*.test.*"`
  - `oracle --dry-run full -p "<task>" --file "src/**"`

- Token 检查：
  - `oracle --dry-run summary --files-report -p "<task>" --file "src/**"`

- 浏览器运行（主路径；长时间运行属正常）：
  - `oracle --engine browser --model gpt-5.2-pro -p "<task>" --file "src/**"`

- 手动粘贴回退：
  - `oracle --render --copy -p "<task>" --file "src/**"`
  - 注意：`--copy` 是 `--copy-markdown` 的隐藏别名。

## 附加文件（`--file`）

`--file` 接受文件、目录和 glob。可多次传入；条目可用逗号分隔。

- 包含：
  - `--file "src/**"`
  - `--file src/index.ts`
  - `--file docs --file README.md`

- 排除：
  - `--file "src/**" --file "!src/**/*.test.ts" --file "!**/*.snap"`

- 默认行为（实现细节）：
  - 默认忽略的目录：`node_modules`、`dist`、`coverage`、`.git`、`.turbo`、`.next`、`build`、`tmp`（除非显式作为字面目录/文件传入，否则跳过）。
  - 展开 glob 时遵循 `.gitignore`。
  - 不跟随符号链接。
  - 点文件默认过滤，除非通过模式显式启用（如 `--file ".github/**"`）。
  - 大于 1 MB 的文件会被拒绝。

## 引擎（API vs 浏览器）

- 自动选择：设置了 `OPENAI_API_KEY` 时用 `api`；否则用 `browser`。
- 浏览器仅支持 GPT + Gemini；Claude/Grok/agent 或多模型运行请用 `--engine api`。
- 浏览器附件：
  - `--browser-attachments auto|never|always`（auto 在约 60k 字符以内内联粘贴，超出则上传）。
- 远程浏览器主机：
  - 主机：`oracle serve --host 0.0.0.0 --port 9473 --token <secret>`
  - 客户端：`oracle --engine browser --remote-host <host:port> --remote-token <secret> -p "<task>" --file "src/**"`

## 会话 + slug

- 存储在 `~/.oracle/sessions` 下（可用 `ORACLE_HOME_DIR` 覆盖）。
- 运行可能脱离或耗时很长（浏览器 + GPT‑5.2 Pro 常见）。若 CLI 超时：不要重跑；重新接入。
  - 列出：`oracle status --hours 72`
  - 接入：`oracle session <id> --render`
- 用 `--slug "<3-5 words>"` 让会话 ID 更易读。
- 存在重复提示词防护；仅在确实需要全新运行时使用 `--force`。

## 提示词模板（高信号）

Oracle 从**零**项目知识开始。假设模型无法推断你的技术栈、构建工具、约定或“显而易见”的路径。需包含：

- 项目简介（技术栈 + 构建/测试命令 + 平台约束）。
- “东西在哪”（关键目录、入口点、配置文件、边界）。
- 确切问题 + 你尝试过的 + 错误文本（原文照录）。
- 约束（“不要改 X”、“必须保留公共 API”等）。
- 期望输出（“返回补丁计划 + 测试”、“给出 3 个方案及权衡”）。

## 安全

- 默认不要附加机密（`.env`、密钥文件、认证 token）。积极脱敏；只分享必要内容。

## “详尽提示词”恢复模式

对于长时间调查，编写独立的提示词 + 文件集，以便数天后重跑：

- 6–30 句项目简介 + 目标。
- 复现步骤 + 确切错误 + 你尝试过的。
- 附加所有需要的上下文文件（入口点、配置、关键模块、文档）。

Oracle 运行是一次性的；模型不记得之前的运行。“恢复上下文”意味着用相同的提示词 + `--file …` 集重跑（或重新接入仍在运行的已保存会话）。