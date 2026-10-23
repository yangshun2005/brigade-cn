---
name: docx
description: 按专业标准创建和编辑 Microsoft Word（.docx）文档——报告、提案、合同、信函、备忘录——支持自定义样式、多级列表、带样式/合并的表格、页眉页脚、目录、超链接、脚注、修订和批注。当用户要求 Brigade 制作、撰写、起草、填写、红线修订或编辑 Word 文档或 .docx 文件时使用。
metadata:
  {
    "brigade":
      {
        "emoji": "📝"
      }
  }
---

# docx — 专业 Word 文档

选择满足需求的最轻量路径。**未至少打开/验证一次结果，绝不可宣称文档完成**（见*验证*）。

| 需求 | 路径 |
|------|------|
| 简单结构化文档（标题、段落、项目符号、基本表格、图片） | **路径 1 — `make_document` 工具** |
| 更丰富的内容（自定义样式、编号/多级列表、合并/底纹表格单元格、页眉页脚、目录、超链接、脚注、行内粗体/斜体/颜色、修订、批注） | **路径 2 — 通过 `brigade exec-node` 编写 `docx` 库脚本** |
| 填写/红线修订现有品牌 `.docx` 且不破坏其样式 | **路径 3 — OOXML 往返** |
| Markdown → 继承企业模板的品牌 Word | **路径 4 — `pandoc`（可选）** |

---

## 路径 1 — 快速结构化文档（`make_document` 工具）

```
make_document(format="docx", content={ title, sections:[{heading, level, paragraphs, bullets, table:{rows}, image:{path}}] })
```
适合快速初稿。它有意受限（单级项目符号、仅字符串表格、无行内格式）。一旦需要更多，转路径 2——不要硬套 Schema。

## 路径 2 — 完整能力：编写 `docx` 库脚本

Brigade 内置 **`docx`** 库（dolanmiu/docx）。编写 CommonJS 脚本并用 **`brigade exec-node`** 运行，它使 Brigade 内置库可在任意位置 `require()`（无需安装）：

1. `write` 一个文件 `gen.cjs`。
2. 运行：`brigade exec-node gen.cjs`

```js
// gen.cjs — 示例；按内容调整
const fs = require("node:fs");
const {
  Document, Packer, Paragraph, TextRun, HeadingLevel, AlignmentType,
  Table, TableRow, TableCell, WidthType, BorderStyle, ShadingType,
  LevelFormat, Header, Footer, PageNumber, TableOfContents, ExternalHyperlink,
} = require("docx");

const doc = new Document({
  styles: { default: { document: { run: { font: "Calibri", size: 22 } } },     // size 为半磅值（22 = 11pt）
    paragraphStyles: [{ id: "Body", name: "Body", run: { size: 22 }, paragraph: { spacing: { after: 160 } } }] },
  numbering: { config: [{ reference: "nums", levels: [{ level: 0, format: LevelFormat.DECIMAL, text: "%1.", alignment: AlignmentType.START }] }] },
  sections: [{
    properties: { page: { size: { width: 12240, height: 15840 } } },           // US Letter，单位 DXA（1440 = 1 英寸）
    headers: { default: new Header({ children: [new Paragraph("Acme Corp — Confidential")] }) },
    footers: { default: new Footer({ children: [new Paragraph({ children: [new TextRun("Page "), new TextRun({ children: [PageNumber.CURRENT] })] })] }) },
    children: [
      new Paragraph({ text: "Q3 Business Review", heading: HeadingLevel.TITLE }),
      new TableOfContents("Contents", { hyperlink: true, headingStyleRange: "1-3" }),
      new Paragraph({ text: "Summary", heading: HeadingLevel.HEADING_1 }),
      new Paragraph({ children: [ new TextRun("Revenue was "), new TextRun({ text: "$1.2M", bold: true }), new TextRun(" — up 18%.") ] }),
      new Paragraph({ text: "First item", numbering: { reference: "nums", level: 0 } }),
      new Paragraph({ children: [ new ExternalHyperlink({ children: [new TextRun({ text: "Full data", style: "Hyperlink" })], link: "https://example.com" }) ] }),
      new Table({
        columnWidths: [4680, 4680],                                            // DXA，总和 = 内容宽度
        rows: [
          new TableRow({ tableHeader: true, children: ["Metric","Value"].map((t) =>
            new TableCell({ width: { size: 4680, type: WidthType.DXA }, shading: { type: ShadingType.CLEAR, fill: "D9E2F3" },
              children: [new Paragraph({ children: [new TextRun({ text: t, bold: true })] })] })) }),
          new TableRow({ children: ["Revenue","$1.2M"].map((t) =>
            new TableCell({ width: { size: 4680, type: WidthType.DXA }, children: [new Paragraph(t)] })) }),
        ],
      }),
    ],
  }],
});
Packer.toBuffer(doc).then((buf) => { fs.writeFileSync(process.argv[2] || "out.docx", buf); console.log("wrote", process.argv[2] || "out.docx"); });
```

`docx` 库还支持：脚注（`FootnoteReferenceRun` + `footnotes`）、批注、**修订**（`InsertedTextRun`/`DeletedTextRun` 带 `author`/`date`）、多节布局、分页符、分栏、内部书签，以及表格单元格内的图片。按需查阅其 API——Word 能表达的一切，它都能生成。

## 路径 3 — 填写/红线修订现有 `.docx`（最高保真度）

要填写品牌公司模板或精准编辑真实文档**而不重新生成**（重新生成会丢失主题），使用 OOXML 往返——解压、编辑 XML、重新压缩。对于纯占位符/文本替换，**`edit_document` 工具**（`replace_text` / `fill_template` / `append`）已能完成，是首选。

对于文本之外的编辑，通过 `brigade exec-node` 用 **`fflate`** 编写脚本：

```js
// edit.cjs — 解压 → 编辑 word/document.xml → 重新压缩（样式/页眉/主题不受影响）
const fs = require("node:fs");
const { unzipSync, zipSync, strToU8, strFromU8 } = require("fflate");
const zip = unzipSync(fs.readFileSync(process.argv[2]));
let xml = strFromU8(zip["word/document.xml"]);
xml = xml.replaceAll("{{client}}", "Acme Corp");          // 填充 <w:t> run 中的占位符
zip["word/document.xml"] = strToU8(xml);
fs.writeFileSync(process.argv[3], zipSync(zip));
```

原始 OOXML 编辑规则：
- 安全：替换 `<w:t>` 内的文本；将 run/段落指向**已定义**的 `styleId`；追加引用现有样式的 `<w:p>`/`<w:r>`。
- **不要**手工添加新部件（图片、超链接目标、页眉）——那还需原子性更新 `[Content_Types].xml` + 匹配的 `_rels/*.rels`，否则 Word 会报损坏。此类情况改用 `docx` 库重新生成。
- 修订是 `<w:ins>`/`<w:del>` 作为 `<w:r>` 的*兄弟节点*（绝不在 run 内）；将原 `<w:rPr>` 复制到变更 run 中；保持编辑最小化，使"全部接受"恰好得到预期文本。作者署名为 "Brigade"。

## 路径 4 — Markdown → 品牌 Word（可选，需 `pandoc`）

对于品牌样式位于企业 `.docx` 的长篇文稿，若已安装 `pandoc`，它会继承所有样式/边距/页眉：
```bash
command -v pandoc >/dev/null 2>&1 && pandoc input.md --reference-doc=brand-template.docx -o out.docx
```
先检测；否则回退到路径 2。`pandoc` 为 GPL——是外部调用的独立程序，绝不内置。

## 使输出专业的约定

- **页面尺寸显式指定**——US Letter `12240×15840` DXA（1440 DXA = 1 英寸）；横向 = 纵向尺寸 + `orientation` 翻转。
- **表格：** 同时给出 `columnWidths`（表格）和每单元格 `width`，单位 **`WidthType.DXA`**（PERCENTAGE 会破坏部分查看器），总和等于内容宽度；用 `ShadingType.CLEAR`（绝不用 SOLID → 黑块）；添加单元格边距；绝不用表格作水平分隔线（用段落底边框）。
- **列表：** 编号/多级列表使用真正的编号配置；绝不粘贴字面 "1." / "•" 字符。
- 优先用项目符号 + 表格而非大段文字；一个 `TITLE`，然后 `HEADING_1..3` 带正确大纲级别（可用的目录必需）。
- 使用排版引号（' ' " "）和短/长破折号。

## 验证（必需）

写入后，确认文件存在且非空。若 LibreOffice 可用，渲染为 PDF 并**查看**——修复溢出/残留占位符，然后重新验证：
```bash
command -v soffice >/dev/null 2>&1 && soffice --headless --convert-to pdf out.docx
```
目录/页码字段在 Word（或 `soffice` 转换）重新计算前显示为空白——若无法运行转换，请告知用户。未至少进行一轮查看并修复，绝不可宣称成功。