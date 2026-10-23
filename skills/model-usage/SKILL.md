---
name: model-usage
description: 使用 CodexBar CLI 本地成本用量数据，汇总 Codex 或 Claude 的各模型用量，包括当前（最近）模型或完整模型明细。当被要求从 codexbar 获取模型级用量/成本数据，或需要从 codexbar 成本 JSON 生成可脚本化的各模型汇总时触发。
metadata:
  {
    "brigade":
      {
        "emoji": "📊",
        "os": ["darwin"],
        "requires": { "bins": ["codexbar"] },
        "install":
          [
            {
              "id": "brew-cask",
              "kind": "brew",
              "formula": "steipete/tap/codexbar",
              "bins": ["codexbar"],
              "label": "安装 CodexBar（brew cask）",
            },
          ],
      },
  }
---

# 模型用量

## 概述

从 CodexBar 本地成本日志获取各模型用量成本。支持 Codex 或 Claude 的“当前模型”（最近一条每日记录）或“全部模型”汇总。

TODO：待 CodexBar CLI 的 Linux 安装路径有文档后，补充 Linux CLI 支持说明。

## 快速开始

1. 通过 CodexBar CLI 获取成本 JSON，或传入 JSON 文件。
2. 使用内置脚本按模型汇总。

```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode current
python {baseDir}/scripts/model_usage.py --provider codex --mode all
python {baseDir}/scripts/model_usage.py --provider claude --mode all --format json --pretty
```

## 当前模型逻辑

- 使用最近一条含 `modelBreakdowns` 的每日记录。
- 选取该记录中成本最高的模型。
- 缺少明细时，回退到 `modelsUsed` 的最后一项。
- 需要指定模型时，用 `--model <name>` 覆盖。

## 输入

- 默认：运行 `codexbar cost --format json --provider <codex|claude>`。
- 文件或标准输入：

```bash
codexbar cost --provider codex --format json > /tmp/cost.json
python {baseDir}/scripts/model_usage.py --input /tmp/cost.json --mode all
cat /tmp/cost.json | python {baseDir}/scripts/model_usage.py --input - --mode current
```

## 输出

- 文本（默认）或 JSON（`--format json --pretty`）。
- 数值仅为各模型成本；CodexBar 输出不按模型拆分 token。

## 参考

- 阅读 `references/codexbar-cli.md` 了解 CLI 参数和成本 JSON 字段。