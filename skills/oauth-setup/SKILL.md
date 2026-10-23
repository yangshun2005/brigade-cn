---
name: oauth-setup
description: 使用内置的 oauth_authorize 工具连接 OAuth 2.0 服务（Gmail、Google API 等）——获取点击即可授权的链接，自动捕获授权码，并封存令牌。只要操作员要求连接 Gmail / Google API / 任何 OAuth 账户，就使用本技能。
metadata:
  {
    "brigade":
      {
        "emoji": "🔐",
      },
  }
---

# OAuth 设置

使用 **`oauth_authorize`** 工具——绝不要在 bash 中手写 `http` 监听器。它会在空闲端口上打开一次性回环监听器（不会出现 EADDRINUSE，无需折腾端口），捕获授权码，并将令牌封存到凭据存储中。

## 需要避免的两个错误（曾因此浪费整个会话）

1. **使用桌面应用类型的 OAuth 客户端**，而非 Web 客户端。在 Google Cloud Console → 凭据 → 创建 OAuth 客户端 ID → **应用类型：桌面应用**。桌面客户端接受任意 `http://127.0.0.1:<port>` 回环重定向，**无需注册重定向 URI**。*Web* 客户端要求预先注册精确的重定向 URI，这正是导致 `redirect_uri_mismatch` 的原因。
2. **使用正确的 scope。** Gmail 发送是 `https://www.googleapis.com/auth/gmail.send`（不是 `https://gmail.com/...`，后者不存在）。如需自动填充账户地址，请添加 `openid https://www.googleapis.com/auth/userinfo.email`。

## 流程

1. 请操作员创建一个**桌面应用**类型的 OAuth 客户端（启用 Gmail API，同意屏幕处于 Testing 状态并将自己设为测试用户），然后粘贴 **client id + secret**。
2. **启动**流程：
   ```
   oauth_authorize({
     action: "start",
     provider: "google-gmail",
     authorizationEndpoint: "https://accounts.google.com/o/oauth2/v2/auth",
     tokenEndpoint: "https://oauth2.googleapis.com/token",
     userInfoEndpoint: "https://www.googleapis.com/oauth2/v3/userinfo",
     clientId: "<id>", clientSecret: "<secret>",
     scopes: ["https://www.googleapis.com/auth/gmail.send", "openid", "https://www.googleapis.com/auth/userinfo.email"],
     extraAuthParams: { access_type: "offline", prompt: "consent" }
   })
   ```
   `access_type=offline` + `prompt=consent` 是获取**刷新令牌**所必需的（否则访问令牌过期后需要重新授权）。
3. 将返回的 `authUrl` 发给操作员点击。
4. **等待**重定向——它会交换授权码并封存令牌：
   ```
   oauth_authorize({ action: "await", flowId: "<flowId from start>" })
   ```
   如果返回状态 `pending`，说明操作员尚未点击——告知对方，然后再次调用 `await`。

## 使用已连接的账户（发送邮件）

令牌封存在凭据存储中——你无法从文件或数据库中读取，也不应尝试。要使用已连接的账户：

1. **查看已连接的内容：** `oauth_authorize({ action: "status" })` → 列出每个账户（provider、email、scopes、过期时间）。不含密钥。
2. **获取可用令牌：** `oauth_authorize({ action: "token", provider: "google-gmail" })` → 返回 `accessToken`。旧令牌过期时，它会用封存的刷新令牌自动刷新，因此你总能拿到有效令牌。（仅在连接了多个账户时才需要传 `provider`。）
3. **调用 API**。Gmail 发送：
   `POST https://gmail.googleapis.com/gmail/v1/users/me/messages/send`，请求头 `Authorization: Bearer <accessToken>`，请求体 `{ "raw": "<base64url RFC-822 message>" }`。

绝不要 `cat` 凭据存储、grep 令牌或手写刷新逻辑——`action:"token"` 是唯一的获取路径，它能保持刷新令牌 + client secret 处于封存状态。