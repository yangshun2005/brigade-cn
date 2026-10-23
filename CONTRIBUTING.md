# 为 Brigade 做贡献

感谢你有兴趣改进 Brigade！本指南介绍如何搭建开发环境、我们遵循的约定，以及变更如何发布。

## 入门

要求：

- **Node.js 22.12 或更新版本**（`node --version`）——运行时使用了 `using` / `AsyncDisposable` 等 v22.12 特性。
- **npm 10+**（随 Node 22 提供）。
- **git**。

```bash
git clone https://github.com/spinabot/brigade.git
cd brigade
npm install
npm run build
```

从你的检出目录运行 CLI：

```bash
npm run dev            # 智能运行器——若 src/ 较新则重新构建 dist/
# 或者，跳过构建步骤：
npm run dev:tsx        # 通过 tsx 直接运行 TypeScript
```

若想在工作时全局使用 `brigade` 可执行文件：

```bash
npm link
brigade --help
```

## 提交 PR 之前

运行完整的本地检查套件——CI 运行相同的命令：

```bash
npm run typecheck      # tsc --noEmit，严格模式
npm test               # node:test，临时目录隔离
npm run build          # 确保生产构建可编译
```

三项必须全部通过。新行为应附带测试，放在对应代码旁边（源文件旁的 `*.test.ts`）。

## 编码约定

- **TypeScript 严格模式**——没有明确理由不得使用 `any` 逃逸。
- **仅 ESM**——相对导入说明符使用 `.js` 扩展名（我们将 TS 编译为 ESM）。
- **与周围代码保持一致**——命名、注释密度和惯用法应与所编辑文件一致。
- **源码中不得包含机密**——绝不提交 API 密钥、令牌、真实电话号码、邮箱或本地绝对路径。配置机密使用 `${VAR_NAME}` 引用，在读取时解析，写入时还原（不持久化解析后的值）。
- **状态保存在 `~/.brigade/` 下**——Brigade 只拥有一个状态目录。不要引入新的全局状态位置。

## 提交信息与发布

Brigade 使用 [Conventional Commits](https://www.conventionalcommits.org/) 配合自动化发布流程：

- `feat:` / `fix:` / `perf:` / `deps:` / `revert:`——触发 Release PR，合并后触发 npm 发布。
- `docs:` / `refactor:` / `test:` / `ci:` / `chore:`——**不**触发发布。

使用清晰、祈使语气的主旨行，例如 `feat(cron): support IANA tz in add`。

## 报告 Bug 与请求功能

使用 Issue 跟踪器中的模板提交 Issue。对于安全漏洞，**不要提交公开 Issue**——参见 [SECURITY.md](SECURITY.md)。

## 彼此友善

保持讨论尊重且切题。假定善意，帮助新人，让项目保持友好。维护者可对辱骂、离题或破坏性的贡献进行管理或移除。

## 联系方式

- **问题与想法**——提交 [GitHub Discussion](https://github.com/spinabot/brigade/discussions) 或发送邮件至 **hello@brigade-agent.ai**。
- **安全问题**——不要提交公开 Issue；参见 [SECURITY.md](SECURITY.md) 或发送邮件至 **security@brigade-agent.ai**。

## 维护者

Brigade 由 **Spinabot** 构建和维护。

- 首席维护者——[@Bhasvanth-Dev9380](https://github.com/Bhasvanth-Dev9380)

我们正有选择地扩充维护者团队。如果你是一位有经验的贡献者，希望帮助塑造 Brigade 的方向——通过代码、文档或社区——我们期待你的来信。成为维护者是一种责任，而非头衔：我们期望持续参与（分类 Issue、审查 PR、推动项目前进）。

有兴趣？发送邮件至 **contributing@brigade-agent.ai**，附上：

- 你在 Brigade 上的 PR 链接（若还没有，这正是起点）。
- 你维护或积极参与的开源项目链接。
- 你的 GitHub 用户名和简短介绍：背景、兴趣领域，以及你实际能投入的时间。

我们会认真审核申请，缓慢而审慎地添加维护者。