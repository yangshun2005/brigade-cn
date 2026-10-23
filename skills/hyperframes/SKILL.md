---
name: hyperframes
description: 编写 HTML 合成内容，由 `render_video` 工具渲染为确定性 MP4 视频——动画图表与仪表盘、数据讲解、文字/引用卡片、动态排版、产品预告片、品牌短视频社交片段。当用户要求 Brigade 根据数据、文本或布局制作、渲染、动画化或生成视频时使用（不适用于照片级真实/AI 素材——那属于 `generate_video`）。
homepage: https://github.com/heygen-com/hyperframes
metadata:
  {
    "brigade":
      {
        "emoji": "🎬",
        "install":
          [
            {
              "id": "npm",
              "kind": "npm",
              "package": "@hyperframes/producer",
              "label": "安装 HyperFrames 渲染引擎（npm）",
            },
          ],
      },
  }
---

# hyperframes — HTML → 确定性 MP4

`render_video` 工具将你编写的 **HTML 合成内容** 渲染为像素级精确的 MP4。无头 Chrome 逐帧推进 **GSAP 主时间线**，每帧捕获一张图像，再由 FFmpeg 编码——因此同一份 HTML 始终产出同一视频。这是处理任何 **程序化、数据驱动** 内容的首选工具；仅当需要照片级真实/AI 生成素材时才使用 `generate_video`。

**适用场景：** 动画柱状图/折线图/环形图、KPI 仪表盘、"关于 X 的 N 个事实"讲解、引用/公告卡片、动态排版、倒计时、前后对比揭示、品牌片头/片尾、竖版社交片段。

## 合成契约（决定渲染成败的关键）

编写一份自包含的 HTML 文档。有两项强制且不直观的要求：

1. **一个暂停的 GSAP 时间线，注册到 `window.__timelines[<composition-id>]`。**
   HyperFrames 通过定位 *这条* 时间线来渲染每一帧。**视频总时长 =
   `tl.duration()`**——你不需要用属性设置时长。
2. **根元素和每个定时元素上的 `data-*` 属性。**

**根合成元素**——必需：`data-composition-id`（必须与 `window.__timelines` 的键一致）、`data-width`、`data-height`、`data-start`；外加 `data-track-index`。

**定时子元素**——每个都需要 `data-start` + `data-track-index`；**可见元素（`<img>`、文本/图形 `<div>`）还需要 `class="clip"`**；`<img>` 还需 `data-duration`。**不要**在根/合成元素上放 `data-duration`——其时长来自时间线。

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <!-- 加载 GSAP（见下方"GSAP + 资源"）。 -->
    <script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>
    <style>
      body { margin: 0; }
      .stage { width: 1080px; height: 1920px; position: relative; overflow: hidden;
               background: #0b0b12; color: #fff; font-family: system-ui, sans-serif; }
      .title { position: absolute; top: 220px; left: 80px; font-size: 96px;
               font-weight: 800; opacity: 0; transform: translateY(24px); }
    </style>
  </head>
  <body>
    <div class="stage"
         data-composition-id="promo"
         data-width="1080" data-height="1920"
         data-start="0" data-track-index="0">
      <div class="title clip" data-start="0" data-duration="4" data-track-index="0">
        Ship faster.
      </div>
    </div>
    <script>
      // 暂停的时间线——HyperFrames 通过逐帧定位来驱动播放。
      const tl = gsap.timeline({ paused: true });
      tl.to(".title", { opacity: 1, y: 0, duration: 0.7, ease: "power2.out" }, 0.4)
        .to(".title", { opacity: 0, duration: 0.5, ease: "power2.in" }, 3.5);
      // 使用与 data-composition-id 相同的 id 注册。这是强制要求。
      window.__timelines = window.__timelines || {};
      window.__timelines["promo"] = tl;
    </script>
  </body>
</html>
```

- TikTok/Reels/Shorts 用竖版 `1080×1920`，横版用 `1920×1080`，
  方形信息流帖子用 `1080×1080`。
- 帧率是引擎默认值（工具不暴露 fps 参数）——按 30fps 平滑运动来编写。

## GSAP + 资源

工具渲染单个独立的 `index.html`（无项目脚手架），因此合成内容必须自带 GSAP 和资源：

- **GSAP**：通过 `<script src="…gsap.min.js">` 加载（如上 CDN）或内联粘贴压缩源码。必须在时间线脚本运行前定义。
- **资源**：将图片/字体嵌入为 `data:` URI，确保不依赖外部文件。
- **确定性**：绝不用 `Date.now()`、`Math.random()`、`setTimeout` 或手写 `requestAnimationFrame` 循环驱动运动——将所有运动放在 GSAP 时间线上。时间线外的动画会与帧步进器失步。

## 工作流

1. **编写合成内容**，遵循上述契约，使用真实数据/文本（绝不用 lorem）。
2. **调用 `render_video`**，参数 `{ html, output_name?, lint? }`。它会执行 lint，在隔离子进程中渲染（真实渲染通常需要一分钟以上），并返回保存的 MP4，格式为 `MEDIA:<path>` 行。
3. **交付**：使用 `send_media({ path })`。
4. 按 `errorType` 处理失败：
   - `composition_invalid`——阅读 lint 消息，修复 HTML，重试。
   - `render_failed`——阅读 stderr 摘录（时间线 id 错误、缺少 GSAP 等）。
   - `render_timeout`——合成内容过重；简化或缩短。
   - `render_unavailable`——引擎/FFmpeg 未安装（见下文）。

## 设计质量（看起来要有意为之，而非套模板）

- 选择契合主题的配色；确定一个强调色，其余保持低调。设定真实的字号层级；给标题加粗并设置字间距。
- 错开揭示（每个元素的时间线位置偏移 0.15–0.3s），让运动看起来像编排过的，而非同时发生。
- 竖版视频中文字距每条边 ≥64px，避免被平台 UI 裁剪。
- 所有运动都加缓动（进入 `power2.out`，退出 `power2.in`）。线性运动显得机械。
- 图表要在时间线上动画化 *数值*（柱高、弧线扫过、计数数字），而非切换静态帧。

## 要求与安装

需要 **Node 22+**、**`@hyperframes/producer`** 引擎（可选依赖——安装后启用视频）以及 PATH 中的 **FFmpeg**。无头 Chromium 由引擎在首次渲染时自动下载（需联网一次）。

```bash
npm i @hyperframes/producer   # 渲染引擎（可选；启用 render_video）
# FFmpeg: brew install ffmpeg | winget install ffmpeg | apt install ffmpeg
```

覆盖项：`BRIGADE_HYPERFRAMES_PATH`（显式 producer 入口文件）、`FFMPEG_PATH`、`BRIGADE_BROWSER_EXECUTABLE` / `PUPPETEER_EXECUTABLE_PATH`。