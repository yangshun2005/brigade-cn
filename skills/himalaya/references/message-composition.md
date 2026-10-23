# 使用 MML（MIME 元语言）撰写邮件

Himalaya 使用 MML 撰写邮件。MML 是一种简单的基于 XML 的语法，可编译为 MIME 消息。

## 基本邮件结构

一封邮件由**头部**列表和**正文**组成，两者之间用空行分隔：

```
From: sender@example.com
To: recipient@example.com
Subject: Hello World

This is the message body.
```

## 头部

常用头部：

- `From`：发件人地址
- `To`：主要收件人
- `Cc`：抄送收件人
- `Bcc`：密送收件人
- `Subject`：邮件主题
- `Reply-To`：回复地址（与 From 不同时使用）
- `In-Reply-To`：所回复邮件的 Message ID

### 地址格式

```
To: user@example.com
To: John Doe <john@example.com>
To: "John Doe" <john@example.com>
To: user1@example.com, user2@example.com, "Jane" <jane@example.com>
```

## 纯文本正文

简单的纯文本邮件：

```
From: alice@localhost
To: bob@localhost
Subject: Plain Text Example

Hello, this is a plain text email.
No special formatting needed.

Best,
Alice
```

## 用 MML 撰写富文本邮件

### 多部分邮件

text/html 替代部分：

```
From: alice@localhost
To: bob@localhost
Subject: Multipart Example

<#multipart type=alternative>
This is the plain text version.
<#part type=text/html>
<html><body><h1>This is the HTML version</h1></body></html>
<#/multipart>
```

### 附件

附加文件：

```
From: alice@localhost
To: bob@localhost
Subject: With Attachment

Here is the document you requested.

<#part filename=/path/to/document.pdf><#/part>
```

自定义附件名称：

```
<#part filename=/path/to/file.pdf name=report.pdf><#/part>
```

多个附件：

```
<#part filename=/path/to/doc1.pdf><#/part>
<#part filename=/path/to/doc2.pdf><#/part>
```

### 内嵌图片

内嵌图片：

```
From: alice@localhost
To: bob@localhost
Subject: Inline Image

<#multipart type=related>
<#part type=text/html>
<html><body>
<p>Check out this image:</p>
<img src="cid:image1">
</body></html>
<#part disposition=inline id=image1 filename=/path/to/image.png><#/part>
<#/multipart>
```

### 混合内容（文本 + 附件）

```
From: alice@localhost
To: bob@localhost
Subject: Mixed Content

<#multipart type=mixed>
<#part type=text/plain>
Please find the attached files.

Best,
Alice
<#part filename=/path/to/file1.pdf><#/part>
<#part filename=/path/to/file2.zip><#/part>
<#/multipart>
```

## MML 标签参考

### `<#multipart>`

将多个部分组合在一起。

- `type=alternative`：同一内容的不同表示形式
- `type=mixed`：独立部分（文本 + 附件）
- `type=related`：相互引用的部分（HTML + 图片）

### `<#part>`

定义邮件部分。

- `type=<mime-type>`：内容类型（如 `text/html`、`application/pdf`）
- `filename=<path>`：要附加的文件
- `name=<name>`：附件的显示名称
- `disposition=inline`：内嵌显示而非作为附件
- `id=<cid>`：用于在 HTML 中引用的 Content ID

## 从 CLI 撰写

### 交互式撰写

打开 `$EDITOR`：

```bash
himalaya message write
```

### 回复（打开编辑器并引用原邮件）

```bash
himalaya message reply 42
himalaya message reply 42 --all  # 回复全部
```

### 转发

```bash
himalaya message forward 42
```

### 从 stdin 发送

```bash
cat message.txt | himalaya template send
```

### 从 CLI 预填头部

```bash
himalaya message write \
  -H "To:recipient@example.com" \
  -H "Subject:Quick Message" \
  "Message body here"
```

## 提示

- 编辑器会打开一个模板；填写头部和正文。
- 保存并退出编辑器即发送；不保存退出则取消。
- 发送时 MML 部分会编译为正确的 MIME。
- 使用 `himalaya message export --full` 查看收到邮件的原始 MIME 结构。