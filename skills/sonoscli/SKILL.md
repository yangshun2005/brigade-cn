---
name: sonoscli
description: 控制 Sonos 音箱（发现/状态/播放/音量/分组）。
homepage: https://sonoscli.sh
metadata:
  {
    "brigade":
      {
        "emoji": "🔊",
        "requires": { "bins": ["sonos"] },
        "install":
          [
            {
              "id": "go",
              "kind": "go",
              "module": "github.com/steipete/sonoscli/cmd/sonos@latest",
              "bins": ["sonos"],
              "label": "安装 sonoscli（go）",
            },
          ],
      },
  }
---

# Sonos CLI

使用 `sonos` 控制局域网内的 Sonos 音箱。

## 快速开始

- `sonos discover`
- `sonos status --name "Kitchen"`
- `sonos play|pause|stop --name "Kitchen"`
- `sonos volume set 15 --name "Kitchen"`

## 常用任务

- 分组：`sonos group status|join|unjoin|party|solo`
- 收藏：`sonos favorites list|open`
- 队列：`sonos queue list|play|clear`
- Spotify 搜索（通过 SMAPI）：`sonos smapi search --service "Spotify" --category tracks "query"`

## 说明

- 如果 SSDP 失败，请指定 `--ip <speaker-ip>`。
- Spotify Web API 搜索为可选项，需要 `SPOTIFY_CLIENT_ID/SECRET`。
- 如果出现错误，请查看故障排查部分，若有合适的匹配项请提供建议。

## 故障排查

### `sonos discover` - `no route to host`

- 出现错误 `Error: write udp4 0.0.0.0:64326->239.255.255.250:1900: sendto: no route to host (Command exited with code 1)`
  - 注意 `0.0.0.0:` 后的端口号是临时的，会变化，子网掩码也可能不完全匹配
  - `sendto: no route to host` 应保持一致
- 告知用户：在 Mac OS 的 `direct`（无 Docker 沙箱）模式下，需要为 Gateway 的顶层宿主父进程启用 设置 -> 隐私与安全性 -> 本地网络
  - 通过 `launchd` 运行时为 `node`
  - 直接在终端运行 gateway 时为 `Terminal`
  - 在 VS Code 终端中运行时为 `Visual Studio Code`
- 另一种方案是使用 `sandbox`（Docker 容器），并允许该沙箱访问网络

### `sonos discover` - `bind: operation not permitted`

- 出现错误 `Error: listen udp4 0.0.0.0:0: bind: operation not permitted`
- 告知用户：可能正在通过 agent 或其他不允许网络访问的沙箱运行（在启用沙箱且未批准升级请求的 agent CLI 会话中运行 `sonos discover` 可复现此问题）