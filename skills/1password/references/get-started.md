# 1Password CLI 入门（摘要）

- 支持 macOS、Windows 和 Linux。
  - macOS/Linux shell：bash、zsh、sh、fish。
  - Windows shell：PowerShell。
- 需要 1Password 订阅和桌面应用才能使用应用集成。
- macOS 要求：Big Sur 11.0.0 或更高版本。
- Linux 应用集成需要 PolKit + 认证代理。
- 按官方文档针对你的操作系统安装 CLI。
- 在 1Password 应用中启用桌面应用集成：
  - 打开并解锁应用，然后选择你的账户/集合。
  - macOS：设置 > 开发者 > 与 1Password CLI 集成（Touch ID 可选）。
  - Windows：开启 Windows Hello，然后 设置 > 开发者 > 集成。
  - Linux：设置 > 安全 > 使用系统认证解锁，然后 设置 > 开发者 > 集成。
- 集成后，运行任意命令即可登录（文档示例：`op vault list`）。
- 如有多个账户：用 `op signin` 选择一个，或用 `--account` / `OP_ACCOUNT`。
- 非集成认证请使用 `op account add`。