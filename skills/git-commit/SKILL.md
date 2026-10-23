---
name: git-commit
description: 干净地暂存、编写和组织 Git 提交与 Pull Request。提交前请阅读本文——了解约定式提交消息格式、何时拆分提交，以及如何用 gh CLI 创建 PR。
requires-bins: git
---

# Git 提交与 PR

提交变更或创建 Pull Request 时使用本文。

## 提交前

- 运行 `git status` 和 `git diff --staged`，确认到底要提交什么。不要盲目 `git add -A`——只暂存属于本次变更的文件。
- 每个提交只包含一个逻辑变更。如果 diff 包含两件不相关的事，就拆成两个提交。
- 不要提交密钥、大型二进制文件或生成的构建产物。检查 `.gitignore` 是否已覆盖它们。

## 消息格式

约定式提交：

```
<type>(<optional scope>): <short imperative summary>

<optional body — what changed and WHY, wrapped at ~72 cols>
```

类型：`feat`、`fix`、`docs`、`refactor`、`test`、`chore`、`perf`、`build`、`ci`。

- 摘要用祈使语气（"add"，而非 "added"），≤72 字符，结尾不加句号。
- 正文说明未来读者需要了解的缘由——不要复述 diff。

## Pull Request（gh CLI）

- `git push -u origin <branch>`，然后 `gh pr create --fill`（或 `--title`/`--body`）。
- PR 正文：改了什么、为什么改、如何验证（运行的测试、手动检查）。
- 创建前确认目标分支正确。

## 需遵循的约定

与仓库现有风格保持一致——先读最近的 `git log --oneline`，如果项目已有格式且与上述不同，就遵循项目格式。