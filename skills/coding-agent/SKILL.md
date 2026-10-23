---
name: coding-agent
description: '通过后台进程将编码任务委派给 Codex、Claude Code 或 Pi 代理。适用场景：(1) 构建/创建新功能或应用，(2) 审查 PR（在临时目录中启动），(3) 重构大型代码库，(4) 需要探索文件的迭代式编码。不适用：简单的一行修复（直接编辑即可）、阅读代码（使用 read 工具）、聊天中绑定线程的 ACP harness 请求（例如在 Discord 线程中启动/运行 Codex 或 Claude Code；请使用 sessions_spawn 并设置 runtime:"acp"），或在 ~/brigade 工作区中进行任何操作（切勿在此启动代理）。Claude Code：使用 --print --permission-mode bypassPermissions（无需 PTY）。Codex/Pi/OpenCode：必须设置 pty:true。'
metadata:
  {
    "brigade":
      {
        "emoji": "🧩",
        "requires": { "anyBins": ["claude", "codex", "opencode", "pi"] },
        "install":
          [
            {
              "id": "node-claude",
              "kind": "node",
              "package": "@anthropic-ai/claude-code",
              "bins": ["claude"],
              "label": "安装 Claude Code CLI (npm)",
            },
            {
              "id": "node-codex",
              "kind": "node",
              "package": "@openai/codex",
              "bins": ["codex"],
              "label": "安装 Codex CLI (npm)",
            },
          ],
      },
  }
---

# 编码代理（bash 优先）

所有编码代理工作均使用 **bash**（可选后台模式）。简单高效。

## ⚠️ PTY 模式：Codex/Pi/OpenCode 需要，Claude Code 不需要

对于 **Codex、Pi 和 OpenCode**，仍需 PTY（交互式终端应用）：

```bash
# ✅ Codex/Pi/OpenCode 的正确用法
bash pty:true command:"codex exec 'Your prompt'"
```

对于 **Claude Code**（`claude` CLI），改用 `--print --permission-mode bypassPermissions`。
PTY 下使用 `--dangerously-skip-permissions` 可能在确认对话框后退出。
`--print` 模式保留完整工具访问权限，并避免交互式确认：

```bash
# ✅ Claude Code 的正确用法（无需 PTY）
cd /path/to/project && claude --permission-mode bypassPermissions --print 'Your task'

# 后台执行：在 exec 工具上使用 background:true

# ❌ Claude Code 的错误用法
bash pty:true command:"claude --dangerously-skip-permissions 'task'"
```

### Bash 工具参数

| 参数         | 类型    | 说明                                                       |
| ------------ | ------- | ---------------------------------------------------------- |
| `command`    | string  | 要运行的 shell 命令                                        |
| `pty`        | boolean | **编码代理必用！** 为交互式 CLI 分配伪终端                 |
| `workdir`    | string  | 工作目录（代理只能看到此文件夹的上下文）                   |
| `background` | boolean | 后台运行，返回 sessionId 用于监控                          |
| `timeout`    | number  | 超时秒数（到期后终止进程）                                 |
| `elevated`   | boolean | 在宿主机而非沙箱中运行（若允许）                           |

### Process 工具操作（用于后台会话）

| 操作        | 说明                                       |
| ----------- | ------------------------------------------ |
| `list`      | 列出所有运行中/最近的会话                  |
| `poll`      | 检查会话是否仍在运行                       |
| `log`       | 获取会话输出（可选 offset/limit）          |
| `write`     | 向 stdin 发送原始数据                      |
| `submit`    | 发送数据 + 换行（相当于输入后按回车）      |
| `send-keys` | 发送按键 token 或十六进制字节              |
| `paste`     | 粘贴文本（可选 bracketed 模式）            |
| `kill`      | 终止会话                                   |

---

## 快速开始：一次性任务

对于快速提示/聊天，创建临时 git 仓库并运行：

```bash
# 快速聊天（Codex 需要 git 仓库！）
SCRATCH=$(mktemp -d) && cd $SCRATCH && git init && codex exec "Your prompt here"

# 或在真实项目中 - 带 PTY！
bash pty:true workdir:~/Projects/myproject command:"codex exec 'Add error handling to the API calls'"
```

**为什么要 git init？** Codex 拒绝在受信任的 git 目录之外运行。创建临时仓库即可解决草稿工作的问题。

---

## 核心模式：workdir + background + pty

对于较长任务，使用后台模式配合 PTY：

```bash
# 在目标目录中启动代理（带 PTY！）
bash pty:true workdir:~/project background:true command:"codex exec --full-auto 'Build a snake game'"
# 返回 sessionId 用于跟踪

# 监控进度
process action:log sessionId:XXX

# 检查是否完成
process action:poll sessionId:XXX

# 发送输入（如果代理提问）
process action:write sessionId:XXX data:"y"

# 带回车提交（相当于输入 "yes" 后按回车）
process action:submit sessionId:XXX data:"yes"

# 需要时终止
process action:kill sessionId:XXX
```

**workdir 为何重要：** 代理在聚焦的目录中启动，不会到处乱读无关文件（比如你的 soul.md 😅）。

---

## Codex CLI

**模型：** `gpt-5.2-codex` 为默认值（在 ~/.codex/config.toml 中设置）

### 标志

| 标志            | 效果                                       |
| --------------- | ------------------------------------------ |
| `exec "prompt"` | 一次性执行，完成后退出                     |
| `--full-auto`   | 沙箱化但在工作区内自动批准                 |
| `--yolo`        | 无沙箱、无审批（最快，也最危险）           |

### 构建/创建

```bash
# 快速一次性（自动批准）- 记得 PTY！
bash pty:true workdir:~/project command:"codex exec --full-auto 'Build a dark mode toggle'"

# 较长任务用后台
bash pty:true workdir:~/project background:true command:"codex --yolo 'Refactor the auth module'"
```

### 审查 PR

**⚠️ 关键：切勿在 Brigade 自己的项目文件夹中审查 PR！**
克隆到临时文件夹或使用 git worktree。

```bash
# 克隆到临时目录以安全审查
REVIEW_DIR=$(mktemp -d)
git clone https://github.com/user/repo.git $REVIEW_DIR
cd $REVIEW_DIR && gh pr checkout 130
bash pty:true workdir:$REVIEW_DIR command:"codex review --base origin/main"
# 完成后清理：trash $REVIEW_DIR

# 或使用 git worktree（保持主分支完整）
git worktree add /tmp/pr-130-review pr-130-branch
bash pty:true workdir:/tmp/pr-130-review command:"codex review --base main"
```

### 批量 PR 审查（并行大军！）

```bash
# 先获取所有 PR refs
git fetch origin '+refs/pull/*/head:refs/remotes/origin/pr/*'

# 部署大军 - 每个 PR 一个 Codex（全部带 PTY！）
bash pty:true workdir:~/project background:true command:"codex exec 'Review PR #86. git diff origin/main...origin/pr/86'"
bash pty:true workdir:~/project background:true command:"codex exec 'Review PR #87. git diff origin/main...origin/pr/87'"

# 监控全部
process action:list

# 将结果发布到 GitHub
gh pr comment <PR#> --body "<review content>"
```

---

## Claude Code

```bash
# 前台
bash workdir:~/project command:"claude --permission-mode bypassPermissions --print 'Your task'"

# 后台
bash workdir:~/project background:true command:"claude --permission-mode bypassPermissions --print 'Your task'"
```

---

## OpenCode

```bash
bash pty:true workdir:~/project command:"opencode run 'Your task'"
```

---

## Pi 编码代理

```bash
# 安装：npm install -g @earendil-works/pi-coding-agent
bash pty:true workdir:~/project command:"pi 'Your task'"

# 非交互模式（仍推荐 PTY）
bash pty:true command:"pi -p 'Summarize src/'"

# 不同提供商/模型
bash pty:true command:"pi --provider openai --model gpt-4o-mini -p 'Your task'"
```

**注意：** Pi 现已启用 Anthropic 提示缓存（PR #584，2026 年 1 月合并）！

---

## 使用 git worktree 并行修复 Issue

并行修复多个 Issue 时，使用 git worktree：

```bash
# 1. 为每个 Issue 创建工作树
git worktree add -b fix/issue-78 /tmp/issue-78 main
git worktree add -b fix/issue-99 /tmp/issue-99 main

# 2. 在每个工作树中启动 Codex（后台 + PTY！）
bash pty:true workdir:/tmp/issue-78 background:true command:"pnpm install && codex --yolo 'Fix issue #78: <description>. Commit and push.'"
bash pty:true workdir:/tmp/issue-99 background:true command:"pnpm install && codex --yolo 'Fix issue #99 from the approved ticket summary. Implement only the in-scope edits and commit after review.'"

# 3. 监控进度
process action:list
process action:log sessionId:XXX

# 4. 修复后创建 PR
cd /tmp/issue-78 && git push -u origin fix/issue-78
gh pr create --repo user/repo --head fix/issue-78 --title "fix: ..." --body "..."

# 5. 清理
git worktree remove /tmp/issue-78
git worktree remove /tmp/issue-99
```

---

## ⚠️ 规则

1. **按代理选择正确的执行模式**：
   - Codex/Pi/OpenCode：`pty:true`
   - Claude Code：`--print --permission-mode bypassPermissions`（无需 PTY）
2. **尊重工具选择** - 用户要求 Codex 就用 Codex。
   - 编排模式：不要自己手写补丁。
   - 如果代理失败/卡住，重新启动它或询问用户方向，但不要悄悄接管。
3. **保持耐心** - 不要因为"慢"就终止会话
4. **用 process:log 监控** - 在不干扰的情况下检查进度
5. **构建用 --full-auto** - 自动批准更改
6. **审查用 vanilla** - 无需特殊标志
7. **并行没问题** - 批量工作时可同时运行多个 Codex 进程
8. **切勿在 Brigade 状态目录中启动 Codex**（`$BRIGADE_STATE_DIR`，默认 `~/.brigade`）- 它会读取你的 soul 文档，对组织架构产生奇怪的想法！
9. **切勿在 ~/Projects/brigade/ 中检出分支** - 那是正在运行的 Brigade 实例！

---

## 进度更新（关键）

在后台启动编码代理时，要让用户了解进展。

- 启动时发送 1 条简短消息（运行什么 + 在哪里）。
- 之后仅在发生变化时更新：
  - 里程碑完成（构建完成、测试通过）
  - 代理提问/需要输入
  - 遇到错误或需要用户操作
  - 代理完成（包含变更内容 + 位置）
- 如果终止了会话，立即说明已终止及原因。

这样可避免用户只看到"Agent failed before reply"而完全不知道发生了什么。

---

## 完成时自动通知

对于长时间运行的后台任务，在提示中附加唤醒触发器，让 Brigade 在代理完成时立即收到通知（而非等待下一次心跳）：

```
... your task here.

When completely finished, run this command to notify me:
brigade system event --text "Done: [brief summary of what was built]" --mode now
```

**示例：**

```bash
bash pty:true workdir:~/project background:true command:"codex --yolo exec 'Build a REST API for todos.

When completely finished, run: brigade system event --text \"Done: Built todos REST API with CRUD endpoints\" --mode now'"
```

这会触发即时唤醒事件——Skippy 几秒内就会被通知，而不是 10 分钟。

---

## 经验总结（2026 年 1 月）

- **PTY 必不可少：** 编码代理是交互式终端应用。没有 `pty:true`，输出会中断或代理会卡住。
- **需要 git 仓库：** Codex 不会在 git 目录之外运行。草稿工作用 `mktemp -d && git init`。
- **exec 是好帮手：** `codex exec "prompt"` 运行后干净退出 - 非常适合一次性任务。
- **submit 与 write：** 用 `submit` 发送输入 + 回车，用 `write` 发送不带换行的原始数据。
- **Sass 有效：** Codex 对俏皮的提示反应良好。让它写一首关于在狮群中当二把手的俳句，得到：_"Second chair, I code / The pride sets the tempo now / Keys glow, I follow"_ 🦁