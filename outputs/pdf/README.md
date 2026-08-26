# outputs/pdf — PDF 产出

按 `shared/format-matrix.md` 的 PDF 渲染路径生成（本环境可靠路径 = matplotlib PdfPages）。

- `*产品说明书.pdf` / `deck-NN.pdf`：16:9 多页 PDF（PPT 同内容渲染稿）。
- `*文档.pdf`：A4 文档 PDF（Word 内容渲染稿）。

## 契约

- **PDF 是渲染稿（不可编辑）**：交付说明必须注明「由 `<源>（.pptx/.docx）` 同内容渲染」。
- 修改内容 → 改源文件（.pptx/.docx）→ 重新渲染 PDF，不直接改 PDF。
- 字体用微软雅黑/SimHei，色值与源文档一致（navy #1F3864 等）。
