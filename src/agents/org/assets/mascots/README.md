# Pride 吉祥物资源

将狮子吉祥物 PNG 放到此处，即可在 `pride-themes.ts` 中启用吉祥物模式模板（即 `mascot: true` 的主题）。

## 存放位置

- **源码**：`src/agents/org/assets/mascots/`（本目录）
- **构建产物**：`dist/agents/org/assets/mascots/`（由 `scripts/build-done.mjs` 复制）
- **不在** `~/.brigade/workspace/` —— 那是用户状态。品牌资源应与代码放在一起，这样 `npm install brigade` 始终会附带它们。

## 文件名约定

渲染器（`pride-html.ts`）默认查找以下文件：

| 文件名                      | 用途                                     |
| --------------------------- | ---------------------------------------- |
| `lion-wave.png`             | 默认吉祥物 —— 挥手的狮子                 |
| `lion-flex.png`             | 备选吉祥物 —— 秀肌肉的狮子               |
| `lion-sit.png`              | 备选吉祥物 —— 坐着的狮子                 |
| `lion-roar.png`             | 备选吉祥物 —— 咆哮的狮子                 |
| `lion-sleep.png`            | 备选吉祥物 —— 睡觉的狮子（用于嘲讽）     |

当主题设置 `mascot: true` 时，渲染器会从可用文件中挑选一个（通过图表的 rng 种子确定性地选择）。若没有 PNG，主题回退到狮子 emoji 🦁。

## 格式

- **PNG**，透明背景（以便在任何主题背景上都能干净呈现）
- 大致为正方形宽高比（渲染器会裁剪/缩放以适配）
- 推荐尺寸：512×512 至 1024×1024（会按模板缩小）
- 以 base64 data URI 嵌入 HTML —— 渲染时不发起网络请求