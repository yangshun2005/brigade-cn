---
name: pdf
description: 按专业标准创建和编辑 PDF——从内容生成、绘制矢量/精确布局、嵌入自定义字体（含 CJK/Unicode）、创建并填写 AcroForm 字段（文本/复选框/单选/下拉）、扁平化、合并/拆分/旋转、盖章/水印、加密、提取文本。当用户要求 Brigade 制作 PDF、填写或构建 PDF 表单，或合并/拆分/盖章/加密 PDF 时使用。
metadata:
  {
    "brigade":
      {
        "emoji": "📄"
      }
  }
---

# pdf — 专业 PDF

| 需求 | 路径 |
|------|------|
| 简单内容 PDF（标题、标题层级、段落、图片） | **路径 1 — `make_document` 工具** |
| 填写现有表单、合并/拆分/盖章/水印 | **路径 2 — `edit_document` 工具** |
| 创建表单字段、矢量绘图、嵌入字体（粗体/CJK）、精确布局、加密、页面操作 | **路径 3 — 通过 `brigade exec-node` 运行脚本 `@cantoo/pdf-lib`** |
| 从 HTML/CSS 生成像素级、品牌风格布局 | **路径 4 — HTML→PDF（可选二进制）** |

> 注意：PDF 是*最终*格式，没有重排引擎。如果接收方需要编辑，请同时交付可编辑源文件（`.docx`/`.xlsx`）**和** PDF。

---

## 路径 1 — 快速内容 PDF（`make_document` 工具）

```
make_document(format="pdf", content={ title, pages:[{ heading, paragraphs, image:{path} }] })
```
流式布局，US-Letter，单栏，自动换行。适合报告/摘要/信函。表单、精确布局或自定义字体 → 路径 3。

## 路径 2 — 编辑现有 PDF（`edit_document` 工具）

- `fill_form {fields}` — 按名称填写现有 AcroForm 字段（文本/复选框/下拉）。
- `merge {pdfs}` / `add_pages {pdfs}` · `split {pages}` · `remove_pages {pages}`。
- `stamp {text}` · `watermark {text}`（45° 半透明）。

**表单判断：** 先检测 PDF 是否有真实可填写字段。有 → 填写字段（干净、类型校验）。如果是扁平/扫描表单（无字段）→ 必须在坐标处绘制文本（路径 3，注释）——绝不要假设字段存在。

## 路径 3 — 完整能力：脚本 `@cantoo/pdf-lib`

Brigade 内置 **`@cantoo/pdf-lib`**（维护中的 pdf-lib 分支——注意带作用域的名称）、**`@pdf-lib/fontkit`**（自定义字体嵌入）和 **`unpdf`**（文本提取）。`write` 一个 `gen.cjs`，运行 `brigade exec-node gen.cjs`：

```js
// gen.cjs — 示例
const fs = require("node:fs");
const { PDFDocument, StandardFonts, rgb } = require("@cantoo/pdf-lib");

(async () => {
  const pdf = await PDFDocument.create();

  // 嵌入自定义字体（真实粗体 / CJK / Unicode）——需要 fontkit
  // const fontkit = require("@pdf-lib/fontkit"); pdf.registerFontkit(fontkit);
  // const font = await pdf.embedFont(fs.readFileSync("Brand-Bold.ttf"));
  const font = await pdf.embedFont(StandardFonts.HelveticaBold);

  const page = pdf.addPage([612, 792]);                 // US Letter；原点在左下角，y 向上
  page.drawText("Invoice", { x: 54, y: 720, size: 22, font, color: rgb(0.1, 0.1, 0.1) });
  page.drawRectangle({ x: 54, y: 700, width: 504, height: 1, color: rgb(0.8, 0.8, 0.8) }); // 矢量线

  // 创建可填写表单字段
  const form = pdf.getForm();
  const name = form.createTextField("client.name");
  name.addToPage(page, { x: 120, y: 640, width: 300, height: 18 });
  const agree = form.createCheckBox("agree");
  agree.addToPage(page, { x: 54, y: 600, width: 14, height: 14 });
  // form.flatten();                                     // 若后续不再填写，将字段烘焙为静态内容

  fs.writeFileSync(process.argv[2] || "out.pdf", await pdf.save());
  console.log("wrote");
})();
```

`@cantoo/pdf-lib` 覆盖：绘制文本/线条/矩形/椭圆/SVG 路径/图片；嵌入自定义 TTF/OTF 字体（子集化，完整 Unicode）；**创建并填写** AcroForm 文本/复选框/单选/下拉/选项列表字段；扁平化；复制/合并/拆分/旋转页面；元数据；**密码加密**（内置此分支的原因）。用 `unpdf` 提取文本/结构。

**坐标陷阱：** pdf-lib 原点在**左下角，y 向上**。如果按左上角图片/规格确定字段位置，需转换：`y_pdf = pageHeight - y_top - height`。每个脚本只用一种坐标约定。

## 路径 4 — 从 HTML+CSS 生成像素级/品牌 PDF（可选）

pdf-lib **没有 HTML/CSS 布局引擎**——要生成设计感、品牌风格文档，请渲染 HTML→PDF。使用现有工具（先检测）：
```bash
command -v soffice >/dev/null 2>&1 && soffice --headless --convert-to pdf brand.html      # LibreOffice 路线
```
无头浏览器（Puppeteer/Playwright `page.pdf()`）若已安装，是更高保真的替代方案。除非品牌设计确实需要 CSS，否则优先路径 1/3。

## 约定

- 填写可填写 PDF 时使用真实 AcroForm 字段；仅对扁平/扫描表单回退到绘制文本注释。
- 超出基本拉丁字符的内容必须嵌入字体——标准 PDF 字体缺少粗体字形及所有 CJK/emoji（缺失字形渲染为方框）。
- 保持 ≥0.5" 边距；对齐网格；不要依赖文本重排（没有引擎——位置由你放置）。

## 验证（必需）

确认输出存在且能打开。对于填写/创建的表单，重新打开并读回字段值（或用 `soffice`/查看器渲染为图片检查位置），然后再宣布完成——绝不盲目填写或放置。