# 发布 Brigade

Brigade 通过自动化的 [release-please](https://github.com/googleapis/release-please) 流程发布到 npm，包名为 [`@spinabot/brigade`](https://www.npmjs.com/package/@spinabot/brigade)。你几乎不需要手动运行 npm 命令。

## 流程

1. **用 [Conventional Commit](https://www.conventionalcommits.org/) 消息将改动合入 `main`**：
   - `feat:` / `fix:` / `perf:` / `deps:` / `revert:` → 计入发布。
   - `docs:` / `refactor:` / `test:` / `ci:` / `chore:` → 不发布。

2. **release-please 自动创建/更新"Release PR"**（见 `.github/workflows/release.yml`）。该 PR 会提升 `package.json` 中的 `version`、更新 `CHANGELOG.md`，并汇总自上次发布以来的所有改动。

3. **想发布时合并 Release PR**。release-please 随后会：
   - 创建 git 标签（如 `v1.12.0`）和 GitHub Release，并
   - 触发 `publish-npm` 任务，依次运行 `npm ci`、`npm run build`、`npm test` 和 `npm publish --access public`（仓库为公开时自动附加 `--provenance`，见下文）。

就这样——新版本即可通过 `npm i @spinabot/brigade` 安装。

## 一次性仓库配置

发布任务需要凭据。**二选一**：

### 方案 A — npm Automation 令牌（当前配置）

1. 在 npmjs.com 创建 **Automation** 访问令牌（Account → Access Tokens → Generate New Token → *Automation*），需具备 `@spinabot` 范围的发布权限。
2. 将其添加为名为 `NPM_TOKEN` 的仓库密钥，任选其一：
   - **CLI：** `gh secret set NPM_TOKEN --repo spinabot/brigade`（在提示符处粘贴令牌——不会回显或记录），或
   - **UI：** Settings → Secrets and variables → Actions → New repository secret。

仓库**公开**后会自动发布 **Provenance**，私有期间则跳过——发布步骤会检测 `github.event.repository.private`，因此切换可见性时无需修改工作流。

### 方案 B — npm Trusted Publishing（OIDC，无需存储令牌）

1. 在 npmjs.com 打开 `@spinabot/brigade` 包设置（首次发布前则为该范围的设置），添加指向 `spinabot/brigade`、工作流为 `release.yml` 的 **trusted publisher**。
2. 从 `.github/workflows/release.yml` 的发布步骤中删除 `NODE_AUTH_TOKEN` 环境变量行。OIDC 的 `id-token: write` 权限和较新的 npm（工作流会升级 npm）即可完成认证，无需密钥。

### 推荐的分支保护

- 保护 `main`；要求合并前通过 `CI` 工作流。
- （可选）添加带必需审核人的 `npm-release` **环境**，让每次发布都经人工批准。工作流已指向该环境。

## 手动发布（应急）

自动化不可用时，在已登录的机器上发布（`npm login`，或在环境中设置 `NPM_TOKEN`）：

```bash
npm ci
npm run build
npm test
npm publish --access public
```

> 此处**不要**加 `--provenance`。Provenance 只能在**公开**仓库的受支持 CI 运行（GitHub Actions OIDC）中生成——该标志在本地机器上会失败。发布工作流会为你附加 provenance。

手动发布后，为保持 release-please 状态一致，请提升 `package.json` 中的 `version`，在 `.release-please-manifest.json` 中设置对应版本，并为该提交打上 `vX.Y.Z` 标签。