---
name: tmux
description: 通过发送按键和抓取窗格输出，远程控制 tmux 会话以操作交互式 CLI。
metadata:
  {
    "brigade":
      {
        "emoji": "🧵",
        "os": ["darwin", "linux"],
        "requires": { "bins": ["tmux"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "tmux",
              "bins": ["tmux"],
              "label": "安装 tmux（brew）",
            },
          ],
      },
  }
---

# tmux 会话控制

通过发送按键和读取输出控制 tmux 会话。管理 Claude Code 会话必备。

## 何时使用

✅ **以下情况使用此技能：**

- 在 tmux 中监控 Claude/Codex 会话
- 向交互式终端应用发送输入
- 抓取 tmux 中长时间运行进程的输出
- 以编程方式切换 tmux 窗格/窗口
- 检查现有会话中的后台工作

## 何时不使用

❌ **以下情况不要使用此技能：**

- 运行一次性 shell 命令 → 直接用 `exec` 工具
- 启动新的后台进程 → 用 `exec` 并设置 `background:true`
- 非交互式脚本 → 用 `exec` 工具
- 进程不在 tmux 中
- 需要创建新的 tmux 会话 → 用 `exec` 执行 `tmux new-session`

## 示例会话

| 会话                    | 用途           |
| ----------------------- | -------------- |
| `shared`                | 主交互会话     |
| `worker-2` - `worker-8` | 并行工作会话   |

## 常用命令

### 列出会话

```bash
tmux list-sessions
tmux ls
```

### 抓取输出

```bash
# 窗格最后 20 行
tmux capture-pane -t shared -p | tail -20

# 全部回滚历史
tmux capture-pane -t shared -p -S -

# 窗口中指定窗格
tmux capture-pane -t shared:0.0 -p
```

### 发送按键

```bash
# 发送文本（不按回车）
tmux send-keys -t shared "hello"

# 发送文本 + 回车
tmux send-keys -t shared "y" Enter

# 发送特殊按键
tmux send-keys -t shared Enter
tmux send-keys -t shared Escape
tmux send-keys -t shared C-c          # Ctrl+C
tmux send-keys -t shared C-d          # Ctrl+D（EOF）
tmux send-keys -t shared C-z          # Ctrl+Z（挂起）
```

### 窗口/窗格导航

```bash
# 选择窗口
tmux select-window -t shared:0

# 选择窗格
tmux select-pane -t shared:0.1

# 列出窗口
tmux list-windows -t shared
```

### 会话管理

```bash
# 创建新会话
tmux new-session -d -s newsession

# 终止会话
tmux kill-session -t sessionname

# 重命名会话
tmux rename-session -t old new
```

## 安全发送输入

对交互式 TUI（Claude Code、Codex 等），将文本和回车分开发送，避免粘贴/多行边界问题：

```bash
tmux send-keys -t shared -l -- "Please apply the patch in src/foo.ts"
sleep 0.1
tmux send-keys -t shared Enter
```

## Claude Code 会话模式

### 检查会话是否需要输入

```bash
# 查找提示符
tmux capture-pane -t worker-3 -p | tail -10 | grep -E "❯|Yes.*No|proceed|permission"
```

### 批准 Claude Code 提示

```bash
# 发送 'y' 和回车
tmux send-keys -t worker-3 'y' Enter

# 或选择编号选项
tmux send-keys -t worker-3 '2' Enter
```

### 检查所有会话状态

```bash
for s in shared worker-2 worker-3 worker-4 worker-5 worker-6 worker-7 worker-8; do
  echo "=== $s ==="
  tmux capture-pane -t $s -p 2>/dev/null | tail -5
done
```

### 向会话发送任务

```bash
tmux send-keys -t worker-4 "Fix the bug in auth.js" Enter
```

## 说明

- 用 `capture-pane -p` 打印到 stdout（脚本化必备）
- `-S -` 抓取全部回滚历史
- 目标格式：`session:window.pane`（如 `shared:0.0`）
- 会话在 SSH 断开后仍保留