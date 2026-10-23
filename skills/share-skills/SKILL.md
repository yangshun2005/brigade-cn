---
name: share-skills
description: 当你需要编写、打包或分享技能文件夹时使用——了解 SKILL.md 格式、技能存放位置，以及如何打包文件夹以便分享给他人。
---

# 分享技能

本技能说明技能的结构，以及如何打包技能以便分享。它是技能编写的配套内容：skill-creator 帮你编写技能，本文则讲如何正确组织并打包交付。

## 技能存放位置

每个技能是 `skills/` 目录下的一个文件夹。文件夹名即技能标识，必须与其 frontmatter 中的 `name` 字段一致（小写）。技能文件夹至少包含一个 `SKILL.md`，也可包含技能引用的辅助文件（脚本、参考文档、资源）。

```
skills/
  my-skill/
    SKILL.md
    reference.md        # 可选辅助文档
    scripts/run.sh      # 可选辅助脚本
```

## SKILL.md 格式

`SKILL.md` 是带 YAML frontmatter 块的 Markdown 文件，后接技能正文。

```yaml
---
name: my-skill          # 必须等于文件夹名，小写
description: Use when ...  # “何时使用”触发指引，不含产品名
# 可选适用性键（不适用则省略）：
os: [darwin, linux]       # 操作系统限制
requires-bins: [foo]      # 必须存在的二进制
requires-any-bins: [a, b] # 至少存在其中一个二进制
requires-env: [MY_TOKEN]  # 必需的环境变量
---
```

正文规范：

- 开头简述技能用途及何时使用。
- 指令须真实：只描述运行时实际能执行的操作。
- 辅助文件用文件夹内的相对路径引用。
- 保持精简——对外呈现的是 description 和元数据，正文按需读取。

## 共享技能的版本管理

打算分享技能时，保留变更日志记录和版本号，便于接收方区分修订。简单做法是在 `SKILL.md` 底部加 `## Changelog` 小节，或放置同级 `CHANGELOG.md`。

## 打包文件夹以便分享

分享技能时，打包整个文件夹（让辅助文件一并携带），以归档形式交付。

```bash
# 在 skills/ 目录下，归档单个技能文件夹
tar -czf my-skill.tar.gz my-skill/
```

接收方将归档解压到自己的 `skills/` 目录：

```bash
tar -xzf my-skill.tar.gz -C ./skills/
```

解压后，确认文件夹名仍与 frontmatter 中的 `name` 一致，且各 `requires-*` 适用性键符合新环境。

## 分享前检查清单

- `name` 等于文件夹名（小写）。
- `description` 已填写且不含产品名。
- 适用性键（`os` / `requires-bins` / `requires-any-bins` / `requires-env`）反映真实需求——无需求则省略。
- 辅助文件位于文件夹内，并以相对路径引用。
- 正文不得指示调用运行时没有的工具。