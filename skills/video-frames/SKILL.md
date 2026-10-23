---
name: video-frames
description: 使用 ffmpeg 从视频中提取帧或短片段。
homepage: https://ffmpeg.org
metadata:
  {
    "brigade":
      {
        "emoji": "🎬",
        "requires": { "bins": ["ffmpeg"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "ffmpeg",
              "bins": ["ffmpeg"],
              "label": "安装 ffmpeg (brew)",
            },
          ],
      },
  }
---

# 视频帧（ffmpeg）

从视频中提取单帧，或生成快速缩略图用于查看。

## 快速开始

第一帧：

```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --out /tmp/frame.jpg
```

指定时间点：

```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --time 00:00:10 --out /tmp/frame-10s.jpg
```

## 说明

- 想了解“这里发生了什么”时，优先用 `--time`。
- 快速分享用 `.jpg`；需要清晰的 UI 帧用 `.png`。