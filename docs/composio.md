# 连接你的应用（Composio）

Brigade 可连接 **1,000+ 外部应用**——Gmail、Slack、GitHub、Notion、Google Calendar、Linear、HubSpot、Jira 等——并代你操作它们（读取邮件、发 Slack 消息、开 GitHub Issue、加日历事件……）。

这通过 **Composio**（composio.dev）实现，该服务负责登录（OAuth）和各应用的对接。你只需提供**一个** Composio API 密钥，之后用自然语言吩咐即可——*“连接我的 Gmail”*、*“给 #team 发条 Slack 消息”*——Brigade 会完成其余工作。

---

## 第 1 步 — 获取 Composio API 密钥

> ⚠️ **最容易出错的地方。** Composio 仪表盘有**两种模式**，切换开关位于左上角 Logo 旁：
>
> | 模式 | 用途 | 用于 Brigade？ |
> |------|------|----------------|
> | **PLATFORM** | “用 Composio SDK 构建和管理 AI 智能体” | ✅ **是——Brigade 需要这个密钥** |
> | **FOR YOU** | “将应用连接到 Claude、Cursor、ChatGPT 等 AI 客户端” | ❌ 否——这里给出的是另一种密钥（供桌面 AI 应用使用），Brigade 无法使用 |
>
> Brigade 基于 Composio **SDK** 构建，因此需要 **PLATFORM** 密钥。**FOR YOU** 提供的是消费级密钥（以 `ck_` 开头），用于将应用接入 Claude/Cursor/ChatGPT——Brigade 会**拒绝**它。PLATFORM 密钥以 `ak_` 开头。

1. 打开 **[dashboard.composio.dev](https://dashboard.composio.dev)** 并登录。
2. 在左上角确认模式开关设为 **PLATFORM**（不是 FOR YOU）。
3. 打开 **Settings → API Keys**，复制你的密钥（形如 `ak_…`）。

日常使用只需从仪表盘获取这一项。

---

## 第 2 步 — 将密钥交给 Brigade

在任意频道告诉你的团队：

> *“这是我的 Composio 密钥：`ak_…`”* —— 或 —— *“把我的 Composio 密钥设为 `ak_…`”*

Brigade 会向 Composio 验证该密钥，并**加密私密存储**——绝不会回显给你，也不会写入聊天记录。只需设置一次。

如果密钥有误，Brigade 会立即告知（*“该密钥被拒绝”*），而不是稍后才失败——这样你就能马上知道是否误用了 FOR YOU 密钥。

---

## 第 3 步 — 连接应用

直接说出你想接入的应用：

> *“连接我的 Gmail。”*  *“接入 Slack。”*  *“连接 GitHub。”*

Brigade 会回复一个**安全登录链接**。点击链接，在常规的服务商页面（Google、Slack、GitHub……）批准访问，即完成连接。随时问 *“连上了吗？”* 即可查看状态。

不确定有哪些可用？问 *“我能连接哪些应用？”*（也可以加 *“……与日历相关的？”*）。列表实时从 Composio 拉取，因此 Composio 后续新增的应用会自动出现——没有任何硬编码。

---

## 第 4 步 — 使用

应用连接后，直接吩咐即可：

> *“把 Q3 数据邮件发给 Sarah。”*  *“把发布说明发到 #announcements。”*
> *“在 acme/web 开一个标题为‘Login bug’的 GitHub Issue。”*  *“我明天日历上有什么？”*

Brigade 会找到正确的操作并执行。

---

## 需要自有登录凭据的应用

Composio 托管大多数应用的登录，因此它们零配置即可连接。少数应用（以及某些套餐下的部分应用）需要**你自己的 OAuth 凭据**。遇到这种情况，Brigade 会告知你，解决办法是在仪表盘做一次性设置：

1. 在 **[dashboard.composio.dev](https://dashboard.composio.dev)**（PLATFORM 模式）打开 **Toolkits**，搜索该应用，然后 **Add to project**。
2. 按提示操作：若有 Composio 默认集成可直接使用，否则粘贴你在该应用开发者网站创建的 OAuth 凭据（仪表盘会给出确切的重定向 URL）。
3. 应用显示为已添加后，再次让 Brigade *“连接它”*——你会收到常规登录链接。

---

## 安全与隐私

- 你的 Composio API 密钥**静态加密**，仅你的 Brigade 可见——绝不回显、记录或发送给模型。
- 连接应用及操作你的账户**仅限所有者**：只有你（操作员）能让 Brigade 连接应用或对其执行操作。
- 应用访问权限保存在**你的** Composio 账户中——你可随时在 Composio 仪表盘查看或撤销任何连接。

---

## 故障排查

| 现象 | 原因 | 解决 |
|------|------|------|
| *“该密钥被拒绝”* | 使用了 **FOR YOU** 密钥（`ck_…`），或密钥输错/被截断 | 从 dashboard.composio.dev → PLATFORM → Settings → API Keys 获取 **PLATFORM** 密钥（`ak_…`） |
| *“未设置 Composio API 密钥”* | 尚未提供密钥 | 告诉 Brigade *“把我的 Composio 密钥设为 `ak_…`”* |
| *“Composio 不为 X 托管受管登录”* | 该应用需要你自己的 OAuth 凭据 | 在仪表盘添加该应用（见上文*需要自有登录凭据的应用*），然后重新连接 |
| 连接链接无法激活 | 尚未在浏览器中完成登录 | 点击链接、批准访问，然后问 *“连上了吗？”* |

---

*面向操作员 / 开发者：* 该集成是常驻、仅限所有者的 `composio` 工具（`src/agents/tools/composio-tool.ts`）。密钥解析顺序：先查密封凭据存储，再查配置中的 `tools.composio.apiKey`，最后查 `COMPOSIO_API_KEY` 环境变量；连接用户 ID 可通过 `tools.composio.userId` 设置（默认 `brigade-owner`）。