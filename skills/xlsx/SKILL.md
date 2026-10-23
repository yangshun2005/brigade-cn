---
name: xlsx
description: 构建和编辑专业 Excel（.xlsx）电子表格、工作簿和简单财务模型——实时公式、数字/日期/货币格式、单元格样式、条件格式、数据验证下拉列表、命名区域、冻结窗格、合并单元格、多工作表。当用户要求 Brigade 制作、填充、更新、建模或编辑电子表格、工作簿或 .xlsx 文件时使用。
metadata:
  {
    "brigade":
      {
        "emoji": "📊"
      }
  }
---

# xlsx — 专业电子表格与模型

| 需求 | 路径 |
|------|------|
| 简单表格 / 多工作表导出（表头 + 行，可选按列数字格式） | **路径 1 — `make_document` 工具** |
| 实时公式、单元格样式、条件格式、下拉列表、命名区域、冻结窗格、合并单元格、日期 | **路径 2 — 通过 `brigade exec-node` 编写 `exceljs` 库脚本** |
| 对现有工作簿做精准编辑 | **路径 3 — `edit_document` 工具** |
| 保证无公式错误 / 获取计算值 | **路径 4 — 重算验证循环（可选 `soffice`）** |

**两条不可妥协的规则**（它们区分真正的模型与粗糙做法）：
1. **用公式，绝不硬编码结果。** 写入 Excel 公式字符串（`=B5*(1+$B$6)`），绝不在代码中计算数字再粘贴字面量——这样输入变化时工作表保持实时。这适用于每个总计、百分比、比率和增长率。
2. **数字就是数字。** 存储 `1200000`，为显示设置格式（`$#,##0`）——绝不用字符串 `"$1.2M"`，否则无法求和或排序。

---

## 路径 1 — 快速表格（`make_document` 工具）

```
make_document(format="xlsx", content={ sheets:[{ name, header, rows, numberFormats }] })
```
单元格可以是 `string | number | {formula, numFmt}`。适合直接的数据表。需要样式、验证、图表或模型 → 路径 2。

## 路径 2 — 完整能力：编写 `exceljs` 库脚本

Brigade 内置 **`exceljs`**。`write` 一个 `gen.cjs`，然后运行 `brigade exec-node gen.cjs`：

```js
// gen.cjs — 示例
const ExcelJS = require("exceljs");
const wb = new ExcelJS.Workbook();
const ws = wb.addWorksheet("Model", { views: [{ state: "frozen", ySplit: 1 }] });   // 冻结表头行

ws.columns = [
  { header: "Item", key: "item", width: 28 },
  { header: "FY24 ($)", key: "v", width: 16, style: { numFmt: "$#,##0" } },
];
ws.getRow(1).font = { bold: true };
ws.getRow(1).fill = { type: "pattern", pattern: "solid", fgColor: { argb: "FFD9E2F3" } };

// 假设块 — 蓝色字体标记硬编码输入（银行家惯例）
ws.getCell("E1").value = "Growth"; ws.getCell("E2").value = 0.18;
ws.getCell("E2").numFmt = "0.0%"; ws.getCell("E2").font = { color: { argb: "FF0000FF" } };
wb.definedNames.add("Model!$E$2", "growth");                                        // 命名区域

ws.addRow({ item: "Revenue", v: 1200000 });
ws.addRow({ item: "Next year", v: { formula: "B2*(1+growth)" } });                  // 公式，引用命名输入
ws.getCell("B3").font = { color: { argb: "FF000000" } };                            // 黑色 = 公式

// 下拉列表 + 条件格式
ws.getCell("A6").dataValidation = { type: "list", allowBlank: false, formulae: ['"Low,Med,High"'] };
ws.addConditionalFormatting({ ref: "B2:B3", rules: [
  { type: "cellIs", operator: "lessThan", formulae: ["0"], style: { font: { color: { argb: "FFFF0000" } } } } ]});

wb.xlsx.writeFile(process.argv[2] || "out.xlsx").then(() => console.log("wrote"));
```

`exceljs` 覆盖：数字/日期格式、字体/填充/边框/对齐、条件格式、数据验证下拉列表、命名区域、冻结窗格、自动筛选、合并单元格、图片、工作表保护、多工作表。日期：传入真实的 `new Date(...)` 并设置日期 `numFmt`（不要传字符串）。

## 路径 3 — 编辑现有工作簿（`edit_document` 工具）

- `set_cells {sheet?, cells:[{ref|row,col, value, numFmt?}]}` — 精准编辑；其他工作表不受影响。
- `append_rows {sheet?, rows}` — 扩展表格。
编辑他人的工作簿时，**严格匹配其现有约定**（列顺序、单位、格式）——模板始终优先于此处的默认值。

## 路径 4 — 重算验证循环（质量保证）

`exceljs` 存储公式**字符串但不求值**——拼写错误（`#REF!`、`#DIV/0!`）在打开文件前不可见。如果存在 LibreOffice，强制重算并检查；否则手动检查区域并保持公式简单。

```bash
command -v soffice >/dev/null 2>&1 && soffice --headless --convert-to pdf --outdir /tmp out.xlsx   # 然后读取 PDF：任何位置都没有 #REF!/#DIV/0!
```
循环：构建 → 重算 → 若出现任何错误标记，修复公式 → 再次重算。目标：交付文件中**零公式错误**。

## 约定（银行级，可选但专业）

- 单元格颜色约定：**蓝色字体 = 硬编码输入**，黑色 = 公式，绿色 = 指向其他工作表的链接，红色 = 指向外部文件的链接；黄色填充 = 关键假设。
- 将假设隔离在独立单元格中，并**绝对**引用（`$B$6`）或通过**命名区域**引用；在单元格批注中记录任何有来源的硬编码（"来源：10-K FY24 第 45 页"）。
- 数字格式：货币 `$#,##0`，单位写在表头；百分比 `0.0%`；倍数 `0.0x`；负数用括号；年份作为文本。

## 验证（必需）

用 `edit_document`/读取重新打开，或确认文件解压后包含预期的工作表名称；如果使用了公式，运行路径 4 的重算并确认零错误后再宣布完成。