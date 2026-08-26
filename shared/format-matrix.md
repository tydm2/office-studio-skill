# 文档格式生成矩阵（format-matrix）

> 用途：orchestrator / word-crafter / slide-designer / sheet-analyst 共用的「格式 → 工具 → 降级」总表。
> 更新：2026-08（多格式支持扩展；依据本机实测环境）。

## 1. 环境探测（生成前必查，不假设工具可用）

| 能力 | 探测方式 | 本环境现状（2026-08） |
|---|---|---|
| python-docx | `import docx` | ✅ 可用 |
| python-pptx | `import pptx` | ❌ 不可用（且沙箱下通常无法联网安装） |
| openpyxl | `import openpyxl` | 需现场探测 |
| matplotlib | `import matplotlib` | ✅ 可用（PDF / 图） |
| LibreOffice | 查 soffice.exe | ❌ 未安装 |
| 无头浏览器打印 | Chrome/Edge `--print-to-pdf` | ⚠️ 沙箱拦截（命名管道），不可靠 |
| 联网 | `pip install` / `Invoke-WebRequest` | ❌ 不可用（pip 超时、网页抓取被拦） |

## 2. 格式 → 主路径 → 降级路径

| 格式 | 主路径 | 降级路径 | 备注 |
|---|---|---|---|
| `.docx`（Word） | python-docx | 手写 OOXML（word-crafter 参考「OOXML 生成可靠性」同源规范） | docx 可用 |
| `.pptx`（PPT） | python-pptx | **复用合法模板骨架**（见 `slide-designer/knowledge/pptx-generation-reliability.md`） | 禁裸手搓最小包 |
| `.xlsx`（Excel） | openpyxl | 手写 OOXML | openpyxl 需探测 |
| `.pdf` | **matplotlib PdfPages**（可靠） | LibreOffice headless 转换 / 无头浏览器（若可用） | 勿用在线转换（无网络） |
| `.md` / `.html` | 直接写文本 | — | — |
| 思维导图 | 文本大纲 / markmap（若可） | 纯文本树 | 视需求 |

## 3. 三条原则

1. **环境优先探测**：生成前 `import` 探测后端，缺什么走什么降级，不假设工具可用。
2. **能编辑才算交付**：Word/PPT/Excel 要求文本原生可编辑；**PDF 是渲染稿（不可编辑）**，交付时须注明「由 `<源>（.pptx/.docx）` 同内容渲染」，便于溯源。
3. **PPT 同内容转 PDF**：用 slide-designer 的 matplotlib 渲染路径（16:9 多页 PDF，同内容同版式），见 `slide-designer/knowledge/pptx-generation-reliability.md` 第 8 节。

## 4. 触发词与路由（orchestrator 用）

- `制作PPT / 幻灯片` → doc_type=ppt → slide-designer（.pptx，必要时附 .pdf 渲染稿）
- `写Word / 文档` → doc_type=word → word-crafter（.docx）
- `做Excel / 表格 / 报表` → doc_type=excel → sheet-analyst（.xlsx）
- `做成PDF / PDF版 / 导出PDF` → doc_type=pdf → 视源内容路由：PPT 内容→slide-designer 渲染；文档→word-crafter 排版后 matplotlib 渲染；无源→按内容直接渲染
- `Markdown / HTML / 思维导图` → 直接由 orchestrator/主代理产出（无专精专家）
