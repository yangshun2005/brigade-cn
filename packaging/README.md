# 打包

**分发和安装 Brigade** 所需的一切都在这里，与 `src/` 中的应用源码分开存放。

## 目录结构

```
packaging/
├── install/                 # 一行命令引导安装器（先装 Node，再装 Brigade）
│   ├── install.sh           # macOS / Linux  →  curl -fsSL https://brigade.spinabot.com/install.sh | sh
│   └── install.ps1          # Windows        →  irm https://brigade.spinabot.com/install.ps1 | iex
│
│   # 计划中 — 每落地一个就加一个：
├── windows/                 # 独立 .exe / Windows 安装程序（Node SEA 或 Inno Setup）
├── macos/                   # .pkg / .app 包 + 公证
├── linux/                   # .deb / .rpm / AppImage
├── homebrew/                # Homebrew formula   (brew install brigade)
└── winget/                  # winget manifest    (winget install brigade)
```

## 安装脚本

`install/install.sh` 和 `install/install.ps1` 会检测 Node；若缺失或版本低于 22.12，则从 nodejs.org 下载最新 Node LTS，然后运行 `npm i -g @spinabot/brigade`。这两个脚本由 **brigade.spinabot.com** 提供，指向本文件夹中的原始文件——因此即使本目录重组，已发布的一行命令仍保持稳定。

## 新增分发目标

1. 在此处创建子文件夹（`windows/`、`homebrew/` 等）。
2. 放入其构建清单/脚本，以及一份简短的 README。
3. 将构建接入 `.github/workflows/` 中的任务，并在主 README 的 Install 章节中引用。