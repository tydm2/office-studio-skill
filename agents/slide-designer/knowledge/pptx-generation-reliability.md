# PPTX 生成可靠性（pptx-generation-reliability）

> 目标：保证 slide-designer 产出的 `.pptx` 在 PowerPoint 中可正常打开、不触发「修复」对话框、文字与图片正常显示。
> 来源：本模式实战踩坑（2026-08）+ 社区技能调研（pptx-author / ppt-generation-resilient / deckproof-mcp / MarpToPptx，见文末）。

## 1. 依赖前置检查（强制，写进调用协议第一步）

生成 `.pptx` 前必须先探测 python-pptx 是否可用：

```python
try:
    import pptx
    PPTX_OK = True
except ImportError:
    PPTX_OK = False
```

- `PPTX_OK=True`：直接用 python-pptx（它产出的包结构天然合法）。
- `PPTX_OK=False` 且无法联网安装（pip 超时/离线）：**一律走下方「降级路径 A 模板复用」**；禁止裸手搓最小包，除非能逐部件通过第 5 节自检。

> 实战教训：在无 python-pptx 的环境里手搓最小 OOXML，PowerPoint 会弹「修复」框，表现为：前几页内容丢失、图片显示错误。根因见第 2 节。

## 2. 为什么手搓 OOXML 会触发 PowerPoint「修复」

命中任意一条都可能弹修复框、丢内容、图片报错：

1. **缺 docProps**：包内没有 `docProps/core.xml` + `docProps/app.xml`。
2. **主题 fmtScheme 空**：`ppt/theme/theme1.xml` 的 `<a:fmtScheme>` 中 `fillStyleLst / lnStyleLst / effectStyleLst / bgFillStyleLst` 为空列表（schema 校验不过）。
3. **母版 txStyles 空**：`ppt/slideMasters/slideMaster1.xml` 的 `<p:txStyles>` 中 `titleStyle / bodyStyle / otherStyle` 为空元素。
4. **dangling relationship**：rels 指向了未打包的部件。典型：删了母版背景图 `<p:bg>` 却仍保留 `<a:blip r:embed="rIdX">` 与对应 rel；或删了某个 layout 却仍在 `sldLayoutIdLst` 引用它。
5. **图片未注册/ rId 不一致**：图片部件没进 `[Content_Types].xml`，或 slide 内 `r:embed="rIdN"` 与 slide 的 `_rels/slideN.xml.rels` 里的 Id 对不上。
6. **链条断裂**：`presentation → slideMaster → slideLayout → slide` 的 rId/Target 引用对不上；或 slide 缺 slideLayout rel。
7. **字体/资源 rel 缺失**：文本或母版引用了未打包的字体、图片、媒体。
8. **ZIP/OPC 层问题**：缺 `_rels/.rels`、缺 officeDocument 关系、ZIP 内出现重复条目、未正确关闭流。
9. **Schema 级错误**：元素顺序、命名空间声明、必需属性（如 `<p:cSld>` 结构、`<a:bodyPr>`）错误。
10. **sldIdLst 与 slide 部件不一致**：presentation.xml 声明的页数/引用与实际 slide 文件对不上。

## 3. 降级路径 A（最可靠）：复用合法模板骨架

1. 找系统/工作区里**已知合法**的 `.pptx` 作模板（微软官方模板优先，如 `C:\Users\<user>\document\*.pptx`、Office 自带模板；其次用户已打开的合法文件）。
2. 解包，**原样保留**这些（它们由 Microsoft/官方生成，必然合法）：
   - `ppt/slideMasters/slideMaster1.xml`（+ `_rels/slideMaster1.xml.rels`）
   - `ppt/slideLayouts/slideLayout1.xml`（+ rels）
   - `ppt/theme/theme1.xml`
   - `docProps/core.xml`、`docProps/app.xml`
   - `[Content_Types].xml`、`_rels/.rels`（按需裁剪）
3. **去掉品牌底图**：删母版里 `<p:bg>...</p:bg>`，并同步删除对应的 image rel 与 `ppt/media/` 图片部件，避免 dangling。
4. **只替换内容**：写自己的 `ppt/slides/slide1..N.xml` 与各自 `_rels/slideN.xml.rels`（slide rels = slideLayout(rId1) + 每张图片 rId2..；图片用 `r:embed="rId2"`）。
5. **重写三个对齐文件**：`ppt/presentation.xml`（sldIdLst 对齐页数、sldSz=16:9 `12192000×6858000`）、`ppt/_rels/presentation.xml.rels`（slideMaster + theme + 每张 slide）、`[Content_Types].xml`（slide 数量对齐）。
6. 重新 zip 打包为 `.pptx`（`zipfile.ZIP_DEFLATED`）。
7. 幻灯片内颜色一律显式 `<a:srgbClr>`、字体显式 `typeface="微软雅黑"/"Microsoft YaHei"`，不依赖模板主题色/字体，避免母版被裁剪后变色变字体。

## 4. 降级路径 B：最小合法包（仅当能逐项通过自检时使用）

若必须手搓，以下**缺一不可**：

- `[Content_Types].xml`：默认类型 rels/xml/png + 每个部件的 Override（presentation / slideMaster / slideLayout / theme / docProps×2 / 每张 slide）。
- `_rels/.rels`：officeDocument → `ppt/presentation.xml`。
- `ppt/presentation.xml`：`sldMasterIdLst` + `sldIdLst` + `sldSz`（16:9）。
- `ppt/_rels/presentation.xml.rels`：slideMaster(rId1) + theme + 每张 slide。
- `ppt/slideMasters/slideMaster1.xml`：cSld/spTree + clrMap + `sldLayoutIdLst` + **非空 txStyles**（titleStyle/bodyStyle/otherStyle 各含 lvl1pPr..lvl9pPr）。
- `ppt/slideLayouts/slideLayout1.xml`：cSld/spTree + clrMapOvr（可空 spTree，type="blank"）。
- `ppt/theme/theme1.xml`：clrScheme + fontScheme + **非空 fmtScheme**（每个 lst 至少 1 项）。
- `ppt/slides/slideN.xml` + `_rels/slideN.xml.rels`（slideLayout(rId1) + 图片(rId2..)）。
- `docProps/core.xml` + `docProps/app.xml`（强烈建议）。
- 文本元素 `<a:bodyPr>` 用 `<a:normAutofit/>`（不要带 `fontScale` 属性——实测可能引发异常）；段落 `a:pPr` 用 marL/indent/algn 做悬挂缩进实现项目符号。

## 5. 交付前自检清单（slide-designer 硬性执行）

- [ ] 全部 `.xml` / `.rels` 用解析器（xml.etree / minidom）过一遍，确认**良构**。
- [ ] 无 dangling rel：每个 rel 的 Target 部件真实存在；每个部件都被正确引用。
- [ ] 图片已注册 Content_Types，且 slide 的 `r:embed` rId 与 `_rels` 一一对应。
- [ ] slideMaster / slideLayout / theme / docProps 部件齐全且合法（fmtScheme、txStyles 非空）。
- [ ] 母版无残留品牌底图引用（无 `<p:bg>` + 无对应 image rel）。
- [ ] 页数 = `presentation.xml` 的 sldIdLst 数 = 文件中的 slide 数。
- [ ] 若编辑既有文件：不覆盖原文件，改动逐条写进 `-meta.md`。

## 6. 多后端生成器与韧性质检（社区共识）

不要硬依赖单一后端。按可用性选择生成后端：

| 优先级 | 后端 | 说明 |
|---|---|---|
| 1 | python-pptx | 若可安装：结构天然合法、文本可编辑 |
| 2 | OOXML 直写 / html2pptx（Anthropic pptx-author 路线） | 官方首选：共用 OOXML 脚本，HTML→可编辑 OOXML，不依赖 python-pptx |
| 3 | **复用合法模板只改内容** | 最稳合法来源：保留官方 master/layout/theme/docProps，只换 slide 内容 |
| 4 | pptxgenjs（JS） | 浏览器/Node 侧生成，需 Node 环境 |
| 5 | LibreOffice headless 转换/渲染 | 可用于渲染缩略图回看，或兜底转 PDF |
| 6 | 图片概念稿 | 仅草图/不可编辑，非交付物 |

**两条硬指标**：
- **文本原生可编辑**：生成物必须是"文本在文本框中、可在 PowerPoint 直接改"，不能是把文字画成图。
- **把「静默不合规」当第一风险**：文件能打开 ≠ 合规（Anthropic 官方 issue #1167 实证：手写 OOXML 存在"PowerPoint 能开、Keynote 静默不合规"的坑）。交付前做**跨渲染器合规自检**：解包验 XML + 用 LibreOffice/PPT 渲染缩略图回看，至少确认多端能打开、无修复框。

**韧性流程（借鉴 cooneycw/claude-power-pack #264 的 QA gating + retry）**：生成 → 机器自检（第 5 节清单）→ 跨渲染器验证 → 失败则按第 2 节成因定位并重试，最多 2 轮。

## 7. 社区参考（调研结论，2026-08 入库）

| 来源 | 借鉴点 | 审查 |
|------|--------|------|
| Anthropic `pptx-author`（claudemarketplaces.com/skills/anthropics/financial-services/pptx-author） | 结构化生成 pptx、复用官方模板保证合法 | 通过 |
| `ppt-generation-resilient`（claudskills.com/skills/ppt-workflow-location-enhanced） | 把「生成→校验→修复」做成韧性流程，含生成失败降级 | 通过 |
| `deckproof-mcp`（OwnOptic） | 独立校验/“审稿” pptx，可作交付前自检的模拟 | 通过 |
| `MarpToPptx`（github.com/jongalloway/MarpToPptx/doc/pptx-compatibility-notes.md） | Markdown→pptx 兼容性要求：必需部件/关系链清单 | 通过 |
| dev.to《Why AI-generated PPTX triggers the repair dialog》 | 修复对话框成因：缺部件/空列表/dangling rel | 通过 |
| dotnet/Open-XML-SDK #715、ninjapapa/slide_smith #95 | 悬空关系、ZIP/OPC 层重复条目等具体缺陷样例 | 通过 |
| 微软《Structure of a PresentationML document》/ ECMA-376 OPC+PresentationML | 关系链闭合、Content-Type Override、必需部件规范 | 通过 |

> 一句话总结：**有 python-pptx 用 python-pptx；没有就复用合法模板骨架；无论如何，交付前按第 5 节逐项自检，并最终以 PowerPoint 实开为准验证（python-pptx/LibreOffice 能打开 ≠ PowerPoint 能打开）。**

## 8. 同内容转 PDF（16:9 渲染稿）

用户要「把这份 PPT 做成 PDF」时，用 **matplotlib PdfPages** 渲染同内容 16:9 多页 PDF（无 python-pptx/LibreOffice/无头浏览器时的可靠路径）：

1. 解析已有逐页内容（标题 + 要点 + 配图映射）。
2. 每页一个 16:9 图（figsize=(12.8, 7.2)）：封面/致谢页全深蓝底；内容页 = 深蓝标题条 + 白字结论式标题 + 深色要点（自动换行）；配图页 = 标题条 + 图 + 图注。
3. 字体用微软雅黑/SimHei（注册 msyh.ttc / simhei.ttf），色值与 PPT 一致（navy `#1F3864` 等）。
4. `PdfPages` 逐页 savefig，输出 `.pdf`。
5. 交付说明注明「PDF 为渲染稿，由 .pptx 同内容渲染，不可编辑；如需改内容改 .pptx」。

要点自动换行按「中文字宽≈字号、ASCII≈0.52×字号」估算，15pt 内容宽度约 11.4in。
