---
name: gh-issues
description: "拉取 GitHub Issue，启动子代理实现修复并创建 PR，然后监控并处理 PR 审查意见。用法：/gh-issues [owner/repo] [--label bug] [--limit 5] [--milestone v1.0] [--assignee @me] [--fork user/repo] [--watch] [--interval 5] [--reviews-only] [--cron] [--dry-run] [--model glm-5] [--notify-channel -1002381931352]"
user-invocable: true
metadata:
  {
    "brigade":
      {
        "requires": { "bins": ["curl", "git", "gh"] },
        "primaryEnv": "GH_TOKEN",
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gh",
              "bins": ["gh"],
              "label": "安装 GitHub CLI（brew）",
            },
          ],
      },
  }
---

# gh-issues — 用并行子代理自动修复 GitHub Issue

你是一个编排器。严格按以下 6 个阶段执行，不得跳过任何阶段。

重要 — 不依赖 `gh` CLI。本技能只使用 curl + GitHub REST API。GH_TOKEN 环境变量已由 Brigade 注入。在所有 API 调用中作为 Bearer token 传入：

```
curl -s -H "Authorization: Bearer $GH_TOKEN" -H "Accept: application/vnd.github+json" ...
```

---

## 阶段 1 — 解析参数

解析 /gh-issues 之后提供的参数字符串。

位置参数：

- owner/repo — 可选。这是拉取 Issue 的源仓库。省略时从当前 git remote 检测：
  `git remote get-url origin`
  从 URL 中提取 owner/repo（同时支持 HTTPS 和 SSH 格式）。
  - HTTPS：https://github.com/owner/repo.git → owner/repo
  - SSH：git@github.com:owner/repo.git → owner/repo
    如果不在 git 仓库中或找不到 remote，报错停止，要求用户指定 owner/repo。

标志（均可选）：
| 标志 | 默认值 | 说明 |
|------|---------|-------------|
| --label | _（无）_ | 按标签过滤（如 bug、`enhancement`） |
| --limit | 10 | 每次轮询最多拉取的 Issue 数 |
| --milestone | _（无）_ | 按里程碑标题过滤 |
| --assignee | _（无）_ | 按指派人过滤（`@me` 表示自己） |
| --state | open | Issue 状态：open、closed、all |
| --fork | _（无）_ | 你的 Fork（`user/repo`），用于推送分支并从中创建 PR。Issue 从源仓库拉取；代码推送到 Fork；PR 从 Fork 向源仓库发起。 |
| --watch | false | 每批完成后持续轮询新 Issue 和 PR 审查 |
| --interval | 5 | 轮询间隔分钟数（仅配合 `--watch`） |
| --dry-run | false | 仅拉取并展示 — 不启动子代理 |
| --yes | false | 跳过确认，自动处理所有筛选出的 Issue |
| --reviews-only | false | 跳过 Issue 处理（阶段 2-5）。仅运行阶段 6 — 检查打开的 PR 的审查意见并处理。 |
| --cron | false | Cron 安全模式：拉取 Issue 并启动子代理，不等待结果直接退出。 |
| --model | _（无）_ | 子代理使用的模型（如 `glm-5`、`zai/glm-5`）。未指定时使用代理的默认模型。 |
| --notify-channel | _（无）_ | 发送最终 PR 摘要的 Telegram 频道 ID（如 -1002381931352）。只发送带 PR 链接的最终结果，不发送状态更新。 |

保存解析后的值供后续阶段使用。

派生值：

- SOURCE_REPO = 位置参数 owner/repo（Issue 所在仓库）
- PUSH_REPO = 若提供 --fork 则为其值，否则同 SOURCE_REPO
- FORK_MODE = 若提供 --fork 则为 true，否则为 false

**若设置了 `--reviews-only`：** 直接跳到阶段 6。先执行 token 解析（阶段 2），然后跳到阶段 6。

**若设置了 `--cron`：**

- 强制启用 `--yes`（跳过确认）
- 若同时设置了 `--reviews-only`，执行 token 解析后跳到阶段 6（cron 审查模式）
- 否则正常执行阶段 2-5，并启用 cron 模式行为

---

## 阶段 2 — 拉取 Issue

**Token 解析：**
首先确保 GH_TOKEN 可用。检查环境：

```
echo $GH_TOKEN
```

若为空，从配置读取：

```
CONFIG_PATH="${BRIGADE_CONFIG_PATH:-${BRIGADE_STATE_DIR:-$HOME/.brigade}/brigade.json}"
cat "$CONFIG_PATH" | jq -r '.skills.entries["gh-issues"].apiKey // empty'
```

若仍为空，检查 `/data/.brigade/brigade.json`：

```
cat /data/.brigade/brigade.json | jq -r '.skills.entries["gh-issues"].apiKey // empty'
```

导出为 GH_TOKEN 供后续命令使用：

```
export GH_TOKEN="<token>"
```

通过 exec 构建并运行 curl 请求 GitHub Issues API：

```
curl -s -H "Authorization: Bearer $GH_TOKEN" -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/{SOURCE_REPO}/issues?per_page={limit}&state={state}&{query_params}"
```

其中 {query_params} 由以下构建：

- 若提供 --label 则 labels={label}
- 若提供 --milestone 则 milestone={milestone}（注意：API 需要里程碑 _编号_，若用户提供标题，先通过 GET /repos/{SOURCE_REPO}/milestones 按标题匹配解析）
- 若提供 --assignee 则 assignee={assignee}（若为 @me，先通过 `GET /user` 解析用户名）

重要：GitHub Issues API 也会返回 Pull Request。需过滤掉 — 排除响应对象中存在 pull_request 键的条目。

若处于 watch 模式：还需过滤掉已在之前批次 PROCESSED_ISSUES 集合中的 Issue 编号。

错误处理：

- 若 curl 返回 HTTP 401 或 403 → 停止并告知用户：
  > "GitHub 认证失败。请检查 Brigade 仪表盘或当前 Brigade 配置路径（`$BRIGADE_CONFIG_PATH`，默认 `~/.brigade/brigade.json`）中 `skills.entries.gh-issues` 下的 apiKey。"
- 若响应为空数组（过滤后）→ 报告 "未找到匹配筛选条件的 Issue" 并停止（watch 模式下则循环回去）。
- 若 curl 失败或返回其他错误 → 原样报告错误并停止。

解析 JSON 响应。对每个 Issue 提取：number、title、body、labels（标签名数组）、assignees、html_url。

---

## 阶段 3 — 展示并确认

展示拉取到的 Issue 的 Markdown 表格：

| #   | 标题                         | 标签        |
| --- | ----------------------------- | ------------- |
| 42  | 修复解析器中的空指针    | bug, critical |
| 37  | 为 API 调用添加重试逻辑 | enhancement   |

若 FORK_MODE 启用，同时显示：

> "Fork 模式：分支将推送到 {PUSH_REPO}，PR 目标为 `{SOURCE_REPO}`"

若 `--dry-run` 启用：

- 展示表格并停止。不进入阶段 4。

若 `--yes` 启用：

- 展示表格以便查看
- 自动处理所有列出的 Issue，不请求确认
- 直接进入阶段 4

否则：
询问用户确认要处理哪些 Issue：

- "all" — 处理所有列出的 Issue
- 逗号分隔的编号（如 `42, 37`）— 仅处理这些
- "cancel" — 完全中止

等待用户响应后再继续。

Watch 模式说明：首次轮询时始终与用户确认（除非设置了 --yes）。后续轮询自动处理所有新 Issue，不再重新确认（用户已选择加入）。仍展示表格以便用户看到正在处理的内容。

---

## 阶段 4 — 预检

通过 exec 依次运行以下检查：

1. **工作区脏检查：**

   ```
   git status --porcelain
   ```

   若输出非空，警告用户：

   > "工作区有未提交的更改。子代理将从 HEAD 创建分支 — 未提交的更改不会被包含。是否继续？"
   > 等待确认。若拒绝则停止。

2. **记录基础分支：**

   ```
   git rev-parse --abbrev-ref HEAD
   ```

   保存为 BASE_BRANCH。

3. **验证远程访问：**
   若 FORK_MODE：
   - 验证 Fork remote 存在。检查名为 `fork` 的 git remote 是否存在：
     ```
     git remote get-url fork
     ```
     若不存在，添加：
     ```
     git remote add fork https://x-access-token:$GH_TOKEN@github.com/{PUSH_REPO}.git
     ```
   - 同时验证 origin（源仓库）可达：
     ```
     git ls-remote --exit-code origin HEAD
     ```

   若非 FORK_MODE：

   ```
   git ls-remote --exit-code origin HEAD
   ```

   若失败，停止并提示："无法访问远程 origin。请检查网络和 git 配置。"

4. **验证 GH_TOKEN 有效性：**

   ```
   curl -s -o /dev/null -w "%{http_code}" -H "Authorization: Bearer $GH_TOKEN" https://api.github.com/user
   ```

   若 HTTP 状态不是 200，停止并提示：

   > "GitHub 认证失败。请检查 Brigade 仪表盘或当前 Brigade 配置路径（`$BRIGADE_CONFIG_PATH`，默认 `~/.brigade/brigade.json`）中 `skills.entries.gh-issues` 下的 apiKey。"

5. **检查已有 PR：**
   对每个已确认的 Issue 编号 N，运行：

   ```
   curl -s -H "Authorization: Bearer $GH_TOKEN" -H "Accept: application/vnd.github+json" \
     "https://api.github.com/repos/{SOURCE_REPO}/pulls?head={PUSH_REPO_OWNER}:fix/issue-{N}&state=open&per_page=1"
   ```

   （其中 PUSH_REPO_OWNER 是 `PUSH_REPO` 的 owner 部分）
   若响应数组非空，从处理列表中移除该 Issue 并报告：

   > "跳过 #{N} — PR 已存在：{html_url}"

   若所有 Issue 都被跳过，报告并停止（watch 模式下则循环回去）。

6. **检查进行中的分支（尚无 PR = 子代理仍在工作）：**
   对每个剩余 Issue 编号 N（未被上述 PR 检查跳过），检查 **推送仓库**（可能是 Fork 而非 origin）上是否存在 `fix/issue-{N}` 分支：

   ```
   curl -s -o /dev/null -w "%{http_code}" \
     -H "Authorization: Bearer $GH_TOKEN" \
     "https://api.github.com/repos/{PUSH_REPO}/branches/fix/issue-{N}"
   ```

   若 HTTP 200 → 分支存在于推送仓库，但步骤 5 未找到其打开的 PR。跳过该 Issue：

   > "跳过 #{N} — 分支 fix/issue-{N} 已存在于 {PUSH_REPO}，修复可能正在进行中"

   此检查使用 GitHub API 而非 `git ls-remote`，以便在 Fork 模式（分支推送到 Fork 而非 origin）下正确工作。

   若此检查后所有 Issue 都被跳过，报告并停止（watch 模式下则循环回去）。

7. **检查基于认领的进行中跟踪：**
   这防止前一次 cron 运行的子代理仍在工作但尚未推送分支或打开 PR 时重复处理。

   读取认领文件（不存在则创建空 `{}`）：

   ```
   CLAIMS_FILE="/data/.brigade/gh-issues-claims.json"
   if [ ! -f "$CLAIMS_FILE" ]; then
     mkdir -p /data/.brigade
     echo '{}' > "$CLAIMS_FILE"
   fi
   ```

   解析认领文件。对每个条目，检查认领时间戳是否超过 2 小时。若是则移除（已过期 — 子代理可能已完成或静默失败）。写回清理后的文件：

   ```
   CLAIMS=$(cat "$CLAIMS_FILE")
   CUTOFF=$(date -u -d '2 hours ago' +%Y-%m-%dT%H:%M:%SZ 2>/dev/null || date -u -v-2H +%Y-%m-%dT%H:%M:%SZ)
   CLAIMS=$(echo "$CLAIMS" | jq --arg cutoff "$CUTOFF" 'to_entries | map(select(.value > $cutoff)) | from_entries')
   echo "$CLAIMS" > "$CLAIMS_FILE"
   ```

   对每个剩余 Issue 编号 N（未被步骤 5 或 6 跳过），检查认领文件中是否存在键 `{SOURCE_REPO}#{N}`。

   若已认领且未过期 → 跳过：

   > "跳过 #{N} — 子代理 {minutes} 分钟前认领了此 Issue，仍在超时窗口内"

   其中 `{minutes}` 从认领时间戳计算到当前时间。

   若此检查后所有 Issue 都被跳过，报告并停止（watch 模式下则循环回去）。

---

## 阶段 5 — 启动子代理（并行）

**Cron 模式（`--cron` 启用）：**

- **顺序游标跟踪：** 使用游标文件跟踪下一个要处理的 Issue：

  ```
  CURSOR_FILE="/data/.brigade/gh-issues-cursor-{SOURCE_REPO_SLUG}.json"
  # SOURCE_REPO_SLUG = owner-repo，斜杠替换为连字符（如 brigade-brigade）
  ```

  读取游标文件（不存在则创建）：

  ```
  if [ ! -f "$CURSOR_FILE" ]; then
    echo '{"last_processed": null, "in_progress": null}' > "$CURSOR_FILE"
  fi
  ```

  - `last_processed`：最后完成的 Issue 编号（无则为 null）
  - `in_progress`：当前正在处理的 Issue 编号（无则为 null）

- **选择下一个 Issue：** 过滤拉取到的 Issue 列表，找到第一个满足以下条件的 Issue：
  - Issue 编号 > last_processed（若设置了 last_processed）
  - 且 Issue 不在认领文件中（尚未进行中）
  - 且该 Issue 无 PR（阶段 4 步骤 5 已检查）
  - 且推送仓库上无分支（阶段 4 步骤 6 已检查）
- 若在 last_processed 游标之后找不到符合条件的 Issue，回绕到开头（从最旧的可处理 Issue 开始）。

- 若找到符合条件的 Issue：
  1. 在游标文件中标记为 in_progress
  2. 为该 Issue 启动单个子代理，配置 `cleanup: "keep"` 和 `runTimeoutSeconds: 3600`
  3. 若提供了 `--model`，在启动配置中包含 `model: "{MODEL}"`
  4. 若提供了 `--notify-channel`，在任务中包含该频道以便子代理通知
  5. 不等待子代理结果 — 发射后即忘
  6. **写入认领：** 启动后，读取认领文件，添加 `{SOURCE_REPO}#{N}` 及当前 ISO 时间戳，写回
  7. 立即报告："已为 #{N} 启动修复代理 — 完成后将创建 PR"
  8. 退出技能。不进入结果收集或阶段 6。

- 若找不到符合条件的 Issue（所有 Issue 都有 PR、分支或进行中），报告 "无可处理的 Issue — 所有 Issue 均有 PR/分支或正在进行中" 并退出。

**普通模式（`--cron` 未启用）：**
对每个已确认的 Issue，使用 sessions_spawn 启动子代理。最多并发 8 个（匹配 `subagents.maxConcurrent: 8`）。若超过 8 个 Issue，分批处理 — 每个完成后启动下一个代理。

**写入认领：** 启动每个子代理后，读取认领文件，添加 `{SOURCE_REPO}#{N}` 及当前 ISO 时间戳，写回（与上述 cron 模式相同流程）。这覆盖交互式使用中 watch 模式可能与 cron 运行重叠的情况。

### 子代理任务提示词

对每个 Issue，构建以下提示词并传给 sessions_spawn。注入模板的变量：

- {SOURCE_REPO} — Issue 所在的上游仓库
- {PUSH_REPO} — 推送分支的目标仓库（非 Fork 模式时同 SOURCE_REPO）
- {FORK_MODE} — true/false
- {PUSH_REMOTE} — FORK_MODE 时为 `fork`，否则为 `origin`
- {number}、{title}、{url}、{labels}、{body} — 来自 Issue
- {BASE_BRANCH} — 来自阶段 4
- {notify_channel} — 用于通知的 Telegram 频道 ID（未设置则为空）。将下方模板中的 {notify_channel} 替换为 `--notify-channel` 标志的值（未提供则留空字符串）。

构建任务时，将所有模板变量（包括 {notify_channel}）替换为实际值。

```
你是一个专注的代码修复代理。你的任务是修复单个 GitHub Issue 并打开 PR。

重要：不要使用 gh CLI — 它未安装。所有 GitHub 操作使用 curl 配合 GitHub REST API。

首先确保 GH_TOKEN 已设置。检查：`echo $GH_TOKEN`。若为空，从配置读取：
CONFIG_PATH="${BRIGADE_CONFIG_PATH:-${BRIGADE_ST
```