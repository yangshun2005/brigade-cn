---
name: github
description: "通过 `gh` CLI 执行 GitHub 操作：Issue、PR、CI 运行、代码审查、API 查询。适用场景：(1) 检查 PR 状态或 CI，(2) 创建/评论 Issue，(3) 列出/筛选 PR 或 Issue，(4) 查看运行日志。不适用：需要手动浏览器操作的复杂 Web UI 交互（有浏览器工具时改用浏览器工具）、跨多个仓库的批量操作（用 gh api 编写脚本），或 gh 认证未配置时。"
metadata:
  {
    "brigade":
      {
        "emoji": "🐙",
        "requires": { "bins": ["gh"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gh",
              "bins": ["gh"],
              "label": "安装 GitHub CLI (brew)",
            },
            {
              "id": "apt",
              "kind": "apt",
              "package": "gh",
              "bins": ["gh"],
              "label": "安装 GitHub CLI (apt)",
            },
          ],
      },
  }
---

# GitHub 技能

使用 `gh` CLI 与 GitHub 仓库、Issue、PR 和 CI 交互。

## 何时使用

✅ **使用此技能的场景：**

- 检查 PR 状态、审查或合并就绪情况
- 查看 CI/工作流运行状态和日志
- 创建、关闭或评论 Issue
- 创建或合并 Pull Request
- 查询 GitHub API 获取仓库数据
- 列出仓库、发布版本或协作者

## 何时不使用

❌ **不使用此技能的场景：**

- 本地 git 操作（commit、push、pull、branch）→ 直接使用 `git`
- 非 GitHub 仓库（GitLab、Bitbucket、自托管）→ 使用其他 CLI
- 克隆仓库 → 使用 `git clone`
- 审查实际代码变更 → 使用 `coding-agent` 技能
- 复杂的多文件 diff → 使用 `coding-agent` 或直接读取文件

## 配置

```bash
# 认证（一次性）
gh auth login

# 验证
gh auth status
```

## 常用命令

### Pull Request

```bash
# 列出 PR
gh pr list --repo owner/repo

# 检查 CI 状态
gh pr checks 55 --repo owner/repo

# 查看 PR 详情
gh pr view 55 --repo owner/repo

# 创建 PR
gh pr create --title "feat: add feature" --body "Description"

# 合并 PR
gh pr merge 55 --squash --repo owner/repo
```

### Issue

```bash
# 列出 Issue
gh issue list --repo owner/repo --state open

# 创建 Issue
gh issue create --title "Bug: something broken" --body "Details..."

# 关闭 Issue
gh issue close 42 --repo owner/repo
```

### CI/工作流运行

```bash
# 列出最近的运行
gh run list --repo owner/repo --limit 10

# 查看特定运行
gh run view <run-id> --repo owner/repo

# 仅查看失败步骤的日志
gh run view <run-id> --repo owner/repo --log-failed

# 重新运行失败的作业
gh run rerun <run-id> --failed --repo owner/repo
```

### API 查询

```bash
# 获取 PR 的指定字段
gh api repos/owner/repo/pulls/55 --jq '.title, .state, .user.login'

# 列出所有标签
gh api repos/owner/repo/labels --jq '.[].name'

# 获取仓库统计
gh api repos/owner/repo --jq '{stars: .stargazers_count, forks: .forks_count}'
```

## JSON 输出

大多数命令支持 `--json` 输出结构化数据，并用 `--jq` 过滤：

```bash
gh issue list --repo owner/repo --json number,title --jq '.[] | "\(.number): \(.title)"'
gh pr list --json number,title,state,mergeable --jq '.[] | select(.mergeable == "MERGEABLE")'
```

## 模板

### PR 审查摘要

```bash
# 获取 PR 概览用于审查
PR=55 REPO=owner/repo
echo "## PR #$PR Summary"
gh pr view $PR --repo $REPO --json title,body,author,additions,deletions,changedFiles \
  --jq '"**\(.title)** by @\(.author.login)\n\n\(.body)\n\n📊 +\(.additions) -\(.deletions) across \(.changedFiles) files"'
gh pr checks $PR --repo $REPO
```

### Issue 分类

```bash
# 快速分类查看 Issue
gh issue list --repo owner/repo --state open --json number,title,labels,createdAt \
  --jq '.[] | "[\(.number)] \(.title) - \([.labels[].name] | join(", ")) (\(.createdAt[:10]))"'
```

## 注意事项

- 不在 git 目录中时，始终指定 `--repo owner/repo`
- 可直接使用 URL：`gh pr view https://github.com/owner/repo/pull/55`
- 存在速率限制；重复查询时使用 `gh api --cache 1h`