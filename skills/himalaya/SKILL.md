---
name: himalaya
description: "通过 IMAP/SMTP 管理邮件的 CLI。使用 `himalaya` 在终端中列出、阅读、撰写、回复、转发、搜索和整理邮件。支持多账户，并可用 MML（MIME Meta Language）撰写邮件。"
homepage: https://github.com/pimalaya/himalaya
metadata:
  {
    "brigade":
      {
        "emoji": "📧",
        "requires": { "bins": ["himalaya"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "himalaya",
              "bins": ["himalaya"],
              "label": "Install Himalaya (brew)",
            },
          ],
      },
  }
---

# Himalaya 邮件 CLI

Himalaya 是一个 CLI 邮件客户端，可通过 IMAP、SMTP、Notmuch 或 Sendmail 后端在终端中管理邮件。

## 参考资料

- `references/configuration.md`（配置文件设置 + IMAP/SMTP 认证）
- `references/message-composition.md`（撰写邮件用的 MML 语法）

## 前置条件

1. 已安装 Himalaya CLI（用 `himalaya --version` 验证）
2. 配置文件位于 `~/.config/himalaya/config.toml`
3. 已配置 IMAP/SMTP 凭据（密码安全存储）

## 配置设置

运行交互式向导设置账户：

```bash
himalaya account configure
```

或手动创建 `~/.config/himalaya/config.toml`：

```toml
[accounts.personal]
email = "you@example.com"
display-name = "Your Name"
default = true

backend.type = "imap"
backend.host = "imap.example.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "you@example.com"
backend.auth.type = "password"
backend.auth.cmd = "pass show email/imap"  # or use keyring

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.example.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "you@example.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.cmd = "pass show email/smtp"
```

## 常用操作

### 列出文件夹

```bash
himalaya folder list
```

### 列出邮件

列出 INBOX（默认）中的邮件：

```bash
himalaya envelope list
```

列出指定文件夹中的邮件：

```bash
himalaya envelope list --folder "Sent"
```

分页列出：

```bash
himalaya envelope list --page 1 --page-size 20
```

### 搜索邮件

```bash
himalaya envelope list from john@example.com subject meeting
```

### 阅读邮件

按 ID 阅读邮件（显示纯文本）：

```bash
himalaya message read 42
```

导出原始 MIME：

```bash
himalaya message export 42 --full
```

### 回复邮件

交互式回复（打开 $EDITOR）：

```bash
himalaya message reply 42
```

全部回复：

```bash
himalaya message reply 42 --all
```

### 转发邮件

```bash
himalaya message forward 42
```

### 撰写新邮件

交互式撰写（打开 $EDITOR）：

```bash
himalaya message write
```

使用模板直接发送：

```bash
cat << 'EOF' | himalaya template send
From: you@example.com
To: recipient@example.com
Subject: Test Message

Hello from Himalaya!
EOF
```

或使用 headers 标志：

```bash
himalaya message write -H "To:recipient@example.com" -H "Subject:Test" "Message body here"
```

### 移动/复制邮件

移动到文件夹：

```bash
himalaya message move 42 "Archive"
```

复制到文件夹：

```bash
himalaya message copy 42 "Important"
```

### 删除邮件

```bash
himalaya message delete 42
```

### 管理标志

添加标志：

```bash
himalaya flag add 42 --flag seen
```

移除标志：

```bash
himalaya flag remove 42 --flag seen
```

## 多账户

列出账户：

```bash
himalaya account list
```

使用指定账户：

```bash
himalaya --account work envelope list
```

## 附件

保存邮件中的附件：

```bash
himalaya attachment download 42
```

保存到指定目录：

```bash
himalaya attachment download 42 --dir ~/Downloads
```

## 输出格式

大多数命令支持 `--output` 输出结构化结果：

```bash
himalaya envelope list --output json
himalaya envelope list --output plain
```

## 调试

启用调试日志：

```bash
RUST_LOG=debug himalaya envelope list
```

带回溯的完整跟踪：

```bash
RUST_LOG=trace RUST_BACKTRACE=1 himalaya envelope list
```

## 提示

- 使用 `himalaya --help` 或 `himalaya <command> --help` 查看详细用法。
- 邮件 ID 相对于当前文件夹；切换文件夹后需重新列出。
- 撰写带附件的富文本邮件时，使用 MML 语法（见 `references/message-composition.md`）。
- 使用 `pass`、系统密钥环或输出密码的命令安全存储密码。