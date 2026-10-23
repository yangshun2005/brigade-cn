# 更新日志

## [1.39.0](https://github.com/spinabot/brigade/compare/brigade-v1.38.0...brigade-v1.39.0) (2026-09-15)


### 新功能

* **team:** 新增持久化协作模式 ([88eb8ca](https://github.com/spinabot/brigade/commit/88eb8ca8e5752e4c7454855b0d4263fb95dff983))
* **team:** 新增持久化协作模式 ([a53879e](https://github.com/spinabot/brigade/commit/a53879e85913729d672600d77918c1bc4b236cfb))


### 问题修复

* **team:** 加固运行时与消息分页 ([afcaca5](https://github.com/spinabot/brigade/commit/afcaca50e70c787c5cc392c29c441a1f94ae282b))

## [1.38.0](https://github.com/spinabot/brigade/compare/brigade-v1.37.1...brigade-v1.38.0) (2026-09-10)


### 新功能

* **providers:** 新增 Token Market ([8021fef](https://github.com/spinabot/brigade/commit/8021fefb3f645135c643fe2bbdd1e2497125b6c5))
* **providers:** 新增 Token Market ([55a872e](https://github.com/spinabot/brigade/commit/55a872e09cb7d621d44cff9e65822375793f2f31))

## [1.37.1](https://github.com/spinabot/brigade/compare/brigade-v1.37.0...brigade-v1.37.1) (2026-09-09)


### 问题修复

* **tideline:** 在可复用引擎边界间保持记忆行为一致 ([0693721](https://github.com/spinabot/brigade/commit/06937210f1aa30b2fc9f34d0696d36bbb4bfd13a))
* **tideline:** 在可复用引擎边界间保持记忆行为一致 ([257dc4f](https://github.com/spinabot/brigade/commit/257dc4f937fe609ce34e0ec051c9c87abd626a30))
* **validation:** 在保存探测证据前约束 provider 数据 ([9175d6b](https://github.com/spinabot/brigade/commit/9175d6b29444d2dc12f202d1bad586e86de094f8))
* **validation:** 在保存探测证据前约束 provider 数据 ([73cc254](https://github.com/spinabot/brigade/commit/73cc2544d661f0ec50dd66cd9916539717cccfe8))

## [1.37.0](https://github.com/spinabot/brigade/compare/brigade-v1.36.1...brigade-v1.37.0) (2026-09-02)


### 新功能

* **connect:** /clear 接受名称参数，并新增 /reset 别名 ([26b2e8b](https://github.com/spinabot/brigade/commit/26b2e8be434eeafe0edcee112499b3157d3a840c))
* **connect:** 新增 /clear，不再把输错的命令发给模型 ([3f94970](https://github.com/spinabot/brigade/commit/3f94970580aeef8e82432288606458026ee99311))
* **usage:** 展示花费的实际去向 ([a09b9b9](https://github.com/spinabot/brigade/commit/a09b9b9680897c35a73d19ac0be9e9299a7f01f5))


### 问题修复

* **cron:** 定时运行从未被计量 ([46b7298](https://github.com/spinabot/brigade/commit/46b7298cff4632b2c6493b8808d239e7b1900042))
* **gateway:** 恢复的会话在首轮之前报告零花费 ([1261319](https://github.com/spinabot/brigade/commit/1261319dc93a5adb32d22119295a5383ad21d7ac))
* **gateway:** 一个会话的上下文用量被画到了其他所有会话上 ([fb3e80a](https://github.com/spinabot/brigade/commit/fb3e80a59c8b3dec801c834c409e0c6cffd1ab93))
* **gateway:** 回收的会话残留了内存行，账单可能记到错误的 agent 上 ([d966f13](https://github.com/spinabot/brigade/commit/d966f13929270c895b185a667bee9371b6980db3))
* **gateway:** subscribe 未经访问检查就泄露了其他 agent 的花费 ([16ea43f](https://github.com/spinabot/brigade/commit/16ea43ff491c0b9cb620d06c73512176a60c1a1d))
* iMessage 回复丢失、花费错误或不可见、一个会话的上下文被画到另一个会话上 ([d6c4558](https://github.com/spinabot/brigade/commit/d6c455865d3bc558db70bf6dba7a6fe8df3ceb79))
* **usage:** 持久化会话花费，而不是从转录重建 ([63315dd](https://github.com/spinabot/brigade/commit/63315dd92901335cee0663526fd9b82c09633cbf))

## [1.36.1](https://github.com/spinabot/brigade/compare/brigade-v1.36.0...brigade-v1.36.1) (2026-08-31)


### 问题修复

* **connect:** 九个斜杠命令在回合进行中不可用 ([04b1cc8](https://github.com/spinabot/brigade/commit/04b1cc82a753154dad9192291f50ee9cb9348502))
* **connect:** 九个斜杠命令在回合进行中不可用 ([9b5b92f](https://github.com/spinabot/brigade/commit/9b5b92f8a424ef8e38258214561dac2f71f40b91))

## [1.36.0](https://github.com/spinabot/brigade/compare/brigade-v1.35.3...brigade-v1.36.0) (2026-08-31)


### 新功能

* **imessage:** agent 思考时显示输入指示器 ([5751c67](https://github.com/spinabot/brigade/commit/5751c67889f6a00519ec0e08b204187177889c06))
* **providers:** 新增 OrcaRouter 作为一等 provider ([e89f76c](https://github.com/spinabot/brigade/commit/e89f76cab07c2306c105c07be0cd331403395ba3))


### 问题修复

* **catalog:** OrcaRouter 条目缺少必需的计费字段 ([f707b68](https://github.com/spinabot/brigade/commit/f707b6844ab37c83ffd9d36702e14e4df15d6815))
* **catalog:** OrcaRouter 条目缺少必需的计费字段 ([e508a76](https://github.com/spinabot/brigade/commit/e508a76350efef0f255bfc3925c713e6f02fdfc7))
* **channels:** 修复四个失效配置 bug，以及其背后的私信/群组误分类 ([ff7a8e5](https://github.com/spinabot/brigade/commit/ff7a8e5ea1519bceefc63d1bf317fc6e3e4b46e3))
* **channels:** 四个失效配置、一处私信/群组误分类，以及缺失的 iMessage 输入指示器 ([c83973f](https://github.com/spinabot/brigade/commit/c83973fa0d47d94692abb1c3d3018ca6c0dc0083))
* **update:** 更新命令每次成功都警告安装过期，且从缓存作答 ([e377d63](https://github.com/spinabot/brigade/commit/e377d63bda2c462621caad78d606ff54b88404d0))
* **update:** 每次成功都警告安装过期，且从缓存作答 ([6973de4](https://github.com/spinabot/brigade/commit/6973de46b70d35f60f45e82e9a545d3f6387de78))

## [1.35.3](https://github.com/spinabot/brigade/compare/brigade-v1.35.2...brigade-v1.35.3) (2026-08-31)


### 问题修复

* **tui:** 统一上下文切换例程、保持中止状态、修复我自己引入的两处回归 ([28f1d17](https://github.com/spinabot/brigade/commit/28f1d1750082eeeb5af98803d3edc4508eb47066))

## [1.35.2](https://github.com/spinabot/brigade/compare/brigade-v1.35.1...brigade-v1.35.2) (2026-08-31)


### 问题修复

* **export:** /export thinking 解析了标志却将其忽略 ([1f333e3](https://github.com/spinabot/brigade/commit/1f333e3b6ea68b82e1d47b5638026cee7471326e))
* 所有 provider 的推理 token，以及真正会重新检查的更新检查 ([ccd9894](https://github.com/spinabot/brigade/commit/ccd98948096bedc7edb339a02117f08ef065407a))
* 所有 provider 的推理 token，以及真正会重新检查的更新检查 ([ecf0243](https://github.com/spinabot/brigade/commit/ecf0243ad564c6739161522d5dd7142d08877134))
* **tui:** /session 和 /agent 限制了输入但未限制视图 ([14a507d](https://github.com/spinabot/brigade/commit/14a507de6fdd645b56e825d4f9516fc6eacb5c78))

## [1.35.1](https://github.com/spinabot/brigade/compare/brigade-v1.35.0...brigade-v1.35.1) (2026-08-31)


### 问题修复

* 一个源文件是二进制文件，且循环保护误判模型在循环 ([477fb09](https://github.com/spinabot/brigade/commit/477fb094df9d1610cc56b716f85ef61a16519be5))
* **tui:** /compact 在空闲时拒绝执行，且头部隐藏了已知信息 ([762ac24](https://github.com/spinabot/brigade/commit/762ac2481a8ae607be54d4fc70fbd7a16cebb5d8))
* **tui:** 说明推理为何无内容，而不是留空 ([6382580](https://github.com/spinabot/brigade/commit/63825806afc5aea8de07bae5938a0e3f5e02c18c))
* **tui:** 展示 TUI 已知的信息——队列、effort、压缩，以及推理为何为空 ([db533c9](https://github.com/spinabot/brigade/commit/db533c9f3c9e4d8c5547b12227dac9371475c4e0))

## [1.35.0](https://github.com/spinabot/brigade/compare/brigade-v1.34.0...brigade-v1.35.0) (2026-08-31)


### 新功能

* 如实展示推理、与 provider 无关的限制、/context ([087fac2](https://github.com/spinabot/brigade/commit/087fac2d9052b71b7d91e378ef8c526962e2265a))
* 如实展示推理、与 provider 无关的限制、/context ([e79a612](https://github.com/spinabot/brigade/commit/e79a61294e78cf7f5f6187dc774ae6473d1270a9))


### 问题修复

* **channels:** 停止宣传运行时从不调用的登出钩子 ([46a15ce](https://github.com/spinabot/brigade/commit/46a15cea8faa55767674420f5c0053886a9ce6a1))
* **config:** 删除失效的并行配置实现 ([46a15ce](https://github.com/spinabot/brigade/commit/46a15cea8faa55767674420f5c0053886a9ce6a1))
* 删除失效的并行配置实现；停止宣传未被调用的登出钩子 ([02867ab](https://github.com/spinabot/brigade/commit/02867ab4bec91603a905c9b143a491721bb09e07))
* 四位审查者认为光标重放设计净收益为负 ([d384279](https://github.com/spinabot/brigade/commit/d38427911558533c595cae626a954c7211483b4e))
* 推理标签可能反转其修复的误述；新增光标重放 ([813e991](https://github.com/spinabot/brigade/commit/813e991cc84537b534caf993a21237b723d6a1b9))

## [1.34.0](https://github.com/spinabot/brigade/compare/brigade-v1.33.0...brigade-v1.34.0) (2026-08-31)


### 新功能

* 回合中压缩、转录工具，以及类型化的 Pi 方言边界 ([960c9ed](https://github.com/spinabot/brigade/commit/960c9edd95686b301c8ba27bdfbe00fe82357a8a))
* 回合中压缩、转录工具，以及类型化的 Pi 方言边界 ([f4b6c6b](https://github.com/spinabot/brigade/commit/f4b6c6bca9b905aec1a60f90747b490de1251463))


### 问题修复

* **documents:** 使用 Unicode 字体时 PDF 生成失败 ([3232a1f](https://github.com/spinabot/brigade/commit/3232a1ffea7a1df0d487a41a14199f9cbf447a91))

## [1.33.0](https://github.com/spinabot/brigade/compare/brigade-v1.32.1...brigade-v1.33.0) (2026-08-27)


### 新功能

* **sessions:** 按会话固定模型，让单个线程运行自己的模型 ([9c7d43d](https://github.com/spinabot/brigade/commit/9c7d43dc3c054ae6cdf158dc2e92342532c6efe3))
* **sessions:** 按会话固定模型，让单个线程运行自己的模型 ([ff63300](https://github.com/spinabot/brigade/commit/ff633004f04c5f140cfcfd69736a0e1c78521ef2))
* **sessions:** 在 sessions.list 行中展示模型固定信息 ([84c3a9c](https://github.com/spinabot/brigade/commit/84c3a9c25f664b1210c46e9e87c5bd6d5bde609f))

## [1.32.1](https://github.com/spinabot/brigade/compare/brigade-v1.32.0...brigade-v1.32.1) (2026-08-22)


### 问题修复

* **sessions:** /delete 接受 /sessions 显示的短键形式 ([b7abf9e](https://github.com/spinabot/brigade/commit/b7abf9e202f1708682e29a84bf0290ba5e428a95))
* **sessions:** /delete 接受 /sessions 显示的短键形式 ([dba8c6c](https://github.com/spinabot/brigade/commit/dba8c6cdc7a4a58ca4ea46a146a2d6dde1c4346b))

## [1.32.0](https://github.com/spinabot/brigade/compare/brigade-v1.31.4...brigade-v1.32.0) (2026-08-22)


### 新功能

* **sessions:** 在 TUI、gateway 和 agent 中命名与删除线程 ([65c8cac](https://github.com/spinabot/brigade/commit/65