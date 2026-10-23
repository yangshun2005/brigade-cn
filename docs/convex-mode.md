# Convex 模式 — 运维手册

Brigade 有两种存储模式。**文件系统模式**（默认）一如既往将所有内容保存在 `~/.brigade/` 下。**Convex 模式**将 Brigade 的全部状态迁移到 Convex 部署中——可自托管在你的机器上，也可使用 Convex Cloud——凭据在离开进程前即已加密。

## Convex 模式下各状态的存放位置

| 状态 | 存放位置 |
|---|---|
| 配置（brigade.json + 备份 + 审计 + 健康检查） | Convex 表 |
| 认证配置（API 密钥——**用你的密钥密封**）、认证/配置状态 | Convex 表 |
| 会话索引 + **完整对话记录** | Convex 表 |
| 执行审批、频道允许列表 + 配对码 | Convex 表 |
| 定时任务 + 运行历史 | Convex 表 |
| 记忆事实 + dream 游标 | Convex 表 |
| Pi 事件日志 + 子系统日志 | Convex 表（批量） |
| 提供商目录（models.json） | 密封 blob + 操作系统缓存镜像 |
| **WhatsApp Baileys 认证**（凭据 + 全部 signal 密钥） | Convex 表（密封） |
| 频道媒体（入站 + 已发送） | 本地缓存热路径 + **后台 Convex 镜像** |
| 工作区（persona MD 文件、**git 仓库**、agent 工作文件） | **本地**（由你选择）+ persona/状态镜像存于 Convex，缺失时恢复 |
| 网关锁 / 浏览器配置 / 组织架构图 / 临时文件 | 操作系统缓存目录（`%LOCALAPPDATA%\Brigade\…`），绝不在 `~/.brigade` |
| `mode.sentinel` | `~/.brigade` 下唯一的引导文件 |

Convex 模式下执行 `rm -rf ~/.brigade` 会丢失：工作区 git 历史 + 临时文件（persona 会在下次启动时从 Convex 恢复），仅此而已。

## 全新开始（首次切换推荐）

```powershell
# 1. 停止所有服务
brigade gateway stop

# 2. （你说过不需要当前数据）清除旧状态
Remove-Item -Recurse -Force ~/.brigade

# 3. 启动自托管 Convex 后端
npm run convex:dev     # 启动本地后端 + 推送 convex/ 函数

# 4. 将 Brigade 指向它并完成引导
$env:BRIGADE_CONVEX_URL = "http://127.0.0.1:3210"
brigade store mode set convex   # 自动创建加密密钥（见下文）
brigade onboard        # 提供商 + 密钥（密钥以密封形式存入 authProfiles）

# 5. 运行
brigade gateway run    # 或直接：brigade
```

**加密密钥——自动生成。** 凭据在离开进程前即用 AES-256-GCM 密封；Convex 永远看不到明文。首次进行 Convex 设置时（`store mode set convex` 或引导向导中选择 Convex），Brigade 会生成一个密钥，仅打印一次（请保存到密码管理器），并存储在操作系统配置位置——Windows 上为 `%LOCALAPPDATA%\Brigade\encryption.key`，macOS 上为 `~/Library/Application Support/brigade/`，Linux 上为 `~/.config/brigade/`——刻意放在 `~/.brigade` 之外，以免清除并恢复流程将其销毁。解析顺序：`BRIGADE_ENCRYPTION_KEY` 环境变量（始终优先）→ 密钥文件 → 关闭。用 `brigade encrypt status` 查看（显示当前生效来源）。

**恢复 vs 全新开始。** 清除 `~/.brigade` 属于恢复——下次启动会从 Convex 恢复所有内容（引导向导也会检测后端中已有的 Brigade，并询问"恢复 / 全新开始"）。真正的全新开始是 `brigade store reset`——它会永久删除后端所有记录、移除模式固定标记，并将旧密钥文件另存为 `.bak`（绝不删除），以便下次引导生成新密钥。

WhatsApp：重新关联一次（`brigade channels link whatsapp`）——signal 密钥现在密封存于 Convex；旧的磁盘认证目录按设计不迁移。此后，即使完全清除 `~/.brigade`，重连也无需重新扫码。

## 验证严格零写入

自动化端到端检查（在后端运行、引导完成后执行）：

```powershell
npm run smoke:strict-zero
# PASS = 每个域都通过 Convex 往返 AND ~/.brigade 保持干净
```


该守卫在 Convex 模式下默认开启（`BRIGADE_STRICT_MODE=warn`）：任何针对 `~/.brigade/` 且不在允许列表（sentinel + 工作区）内的写入，都会连同堆栈被大声记录。待一次会话运行干净后，可切换为强制模式：

```powershell
$env:BRIGADE_STRICT_MODE = "enforce"   # 违规现在会抛出异常
```

留意网关输出中的 `STRICT-ZERO VIOLATION` 行；正常运行时不应出现。

## 模式切换

- `brigade store mode show` — 当前模式
- `brigade store mode set filesystem` — 切回文件模式（即使 Convex 后端宕机也可用；诊断命令从不依赖后端）
- sentinel 固定该选择；`BRIGADE_MODE` 环境变量可覆盖以进行一次性诊断（`BRIGADE_FORCE_MODE=1` 可绕过不匹配拒绝）。

## 密钥轮换

```powershell
$env:BRIGADE_ENCRYPTION_KEY_OLD = $env:BRIGADE_ENCRYPTION_KEY
$env:BRIGADE_ENCRYPTION_KEY = "<new 64-char hex>"
# 读取时先试新密钥，再试旧密钥——可从容轮换，待所有内容重新密封后
# 再删除 OLD 变量。
```

## 延迟表现

- 热路径上的读取（配置、会话索引、审批门、记忆、Baileys signal 密钥）由进程内缓存提供，缓存在启动时通过一轮并行操作完成水合——每轮开销与文件系统模式相同。
- 写入会同步预热缓存（下次读取立即可见），并通过有序后台链刷新到 Convex；网关在关闭时排空所有链。
- 媒体发送从本地磁盘流式传输（零额外延迟）；Convex 副本在后台上传。
- 对话记录批量处理（50 条记录 / 250 ms），并在每轮结束时排空。