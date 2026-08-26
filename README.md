# office-studio — Multi-Agent Workflow for Office Documents

**[English](README.md)** · [简体中文](README.zh-CN.md) · [Español](README.es.md) · [Français](README.fr.md) · [Русский](README.ru.md) · [日本語](README.ja.md)



> Target platform: **DSH** (this environment). One pipeline for efficiently creating/editing three kinds of office documents — **Word · PPT · Excel** — with quality-first results and support for **one-click switching between multiple styles**.

## I. Pipeline

```
【新建】用户一句话（如"帮我做一份商务风PPT"）
        │
        ▼
orchestrator（大脑·策划调度）──澄清需求+选风格──▶ 产出 outputs/plans/plan-NN.md ──▶ 用户确认
        │                                              │
        │ 派发自包含任务包（含已选风格）                │
        ├──────────────┬──────────────┬────────────────┤
        ▼              ▼              ▼                ▼
   word-crafter   slide-designer  sheet-analyst   （按 doc_type 只派一个）
   （Word撰写·小组） （PPT设计·小组） （Excel报表·单一资深）
        ▼              ▼              ▼
   outputs/word/   outputs/ppt/    outputs/excel/
   doc-NN.docx     deck-NN.pptx    sheet-NN.xlsx
        └──────────────┴──────────────┴────────────────┘
                        │ 汇总汇报
                        ▼
                    orchestrator → 交付用户 → 迭代（send_message 续聊修订）

【编辑】拖入/提供现有文档（.docx/.pptx/.xlsx 路径）
        │
        ▼
orchestrator ──读取识别（结构/风格/内容概要）──▶ 澄清 修改目标/保留项/风格
        │
        ▼ 产出 outputs/plans/edit-NN.md（source_file + change_request + keep）
   对应专家：先读原文件 → 增量修改（保留原结构风格）→ 产出 doc/deck/sheet-NN + 逐条改动清单
        │
        ▼
   交付用户 → 迭代（send_message 续聊修订）
```

## II. Trigger-Word Mapping

| Agent | One-line responsibility | Trigger words (natural language, including synonymous variants) |
|--------|-----------|------------------------------|
| **orchestrator** (Brain) | Takes a requirement → outputs the selected style + self-contained task package → aggregates delivery | New: `开始策划` `新建文档` `帮我做一份PPT` `帮我写一份Word` `帮我做一份Excel` `做文档`; Edit: `编辑文档` `修改文档` `改这份PPT` `改这个Word` (prefers the edit branch when the user provides an existing file) |
| **word-crafter** | Takes plan + style → outputs a spec-compliant .docx | New: `撰写Word` `写正文` `生成Word`; Edit: `改Word` `修改文档` `编辑这份Word` |
| **slide-designer** | Takes plan + style → outputs a narrative .pptx | New: `制作PPT` `做幻灯片` `生成PPT`; Edit: `改PPT` `修改这份PPT` |
| **sheet-analyst** | Takes data + plan + style → outputs a conclusion-first .xlsx | New: `制作Excel` `做报表` `生成Excel` `分析数据`; Edit: `改Excel` `修改表格` `编辑这份Excel` |

**Trigger rules**: New tasks (no corresponding plan archive) always enter `orchestrator` first for clarification; after `plan-NN.md` is confirmed by the user, use each expert's dedicated trigger words to go directly. Trigger words do not conflict with one another, nor with built-in system commands.

## III. Directory Structure

```
office-studio/
  README.md                      # 本文件：导航 + 流水线图 + 触发词映射
  shared/
    style-catalog.md             # 刷新型风格库（14 种风格：6 经典 + 8 社区借鉴，多风格切换核心）
    style-switcher.md            # 风格选择器 + 不满意换一轮协议
    brand-kit.md                 # ★品牌套件（v2）：跨文档品牌统一——同一客户的 Word/PPT/Excel 共用品牌色/字体/Logo/页眉页脚
  blueprints/
    办公文档.md                    # 领域拓扑沉淀（复用与决策记录，v1.6 机制）
  feedback-log.md                # 工作流级需求记忆（运行期迭代用）
  usage-log.md                   # 工作流级使用留痕（运行期迭代用）
  agents/
    orchestrator/                # 大脑（策划/调度）
      AGENT.md
      knowledge/                 # task-intake / quality-redlines / community-refs
      references/                # feedback-log / usage-log
    word-crafter/                # Word 撰写专家（专家小组制）
      AGENT.md
      knowledge/                 # craft-methodology / style-application / quality-checklist / output-template / community-refs
      references/
    slide-designer/              # PPT 设计专家（专家小组制）
      AGENT.md
      knowledge/                 # narrative-methodology / style-application / quality-checklist / output-template / community-refs / visual-qa / plugin-ecosystem
      scripts/visual-qa.py       # 视觉审稿质检脚本（溢出/重叠/对比度/越界/字体）
      references/
    sheet-analyst/               # Excel 报表专家（单一资深专家）
      AGENT.md
      knowledge/                 # analysis-methodology / style-application / quality-checklist / output-template / community-refs
      references/
  outputs/
    plans/                       # 策划任务包 plan-NN.md / 编辑任务包 edit-NN.md
    word/                        # doc-NN.docx + doc-NN-meta.md
    ppt/                         # deck-NN.pptx + deck-NN-meta.md
    excel/                       # sheet-NN.xlsx + sheet-NN-meta.md
```

## IV. File Contracts (upstream output → downstream reads)

1. **Planning → each expert (new)**: `outputs/plans/plan-NN.md`, with a fixed header metadata format:
   ```yaml
   ---
   doc_id: plan-NN
   title: <title>
   doc_type: word | ppt | excel
   style_id: <see shared/style-catalog.md — 14 styles: 6 classic + 8 community-inspired; selector in shared/style-switcher.md>
   brand_kit: <optional; write brand-NN when the user provides brand assets, rules in shared/brand-kit.md; omit if none>
   audience: <audience>
   purpose: <purpose>
   length: <length>
   data_source: <optional, Excel data file path>
   ---
   ```
1b. **Edit task → each expert (edit)**: `outputs/plans/edit-NN.md`:
   ```yaml
   ---
   doc_id: edit-NN
   source_file: <original document path, provided/dragged in by the user>
   doc_type: word | ppt | excel
   mode: edit
   change_request: <change request, specific to paragraph/page/cell>
   keep: <must-keep items: original format / existing conclusions / data>
   style_id: <defaults to the original style; switching styles requires explicit user specification>
   ---
   ```
2. **Each expert's output**: `outputs/<type>/<filename>`, accompanied by a same-named `-meta.md` (recording: doc_id, style_id, **brand_kit (if any)**, generation date, data source, change notes).
3. **Iteration**: All user revision comments go through DSH `send_message` to continue the atomic agent's conversation for revision, preserving context; revisions must be written into the "change notes" of `-meta.md`, never silently rewriting confirmed content.
4. **Brand kit (v2)**: When the user provides brand assets (brand colors / Logo / fonts / templates) → orchestrator collects them into `shared/brand-kit.md`, the plan writes `brand_kit: brand-NN`, and all three experts apply them uniformly (brand takes priority over the style catalog); when delivering multiple documents under the same brand_id, orchestrator verifies brand consistency (see `shared/brand-kit.md`).

## V. Community Credits (design references)

| Source | Borrowed points |
|------|--------|
| [anthropics/skills](https://github.com/anthropics/skills) (official) | The docx/pptx/xlsx "read existing file → generate → verify openable" paradigm |
| [johnson7788/skill-ppt-agents](https://github.com/johnson7788/skill-ppt-agents) + [MultiAgentPPT](https://github.com/johnson7788/MultiAgentPPT) | PPT "outline → content → design" multi-stage, with independent acceptance at each stage |
| [rafalozan0/DocFlow](https://github.com/rafalozan0/DocFlow-Presentations-and-Docs-Skill) | Unified python-pptx/docx/openpyxl for the trio + file-based template directory (multi-style switching) |
| [cabbage2000-lab/data-analysis-skills](https://github.com/cabbage2000-lab/data-analysis-skills) | Excel conclusion-first six-paragraph structure, traceable data, no fabrication, correlation ≠ causation |
| [fleurytian/awesome-claude-skills](https://github.com/fleurytian/awesome-claude-skills) (ex-McKinsey) | Pyramid Principle / MECE narrative structure |
| [hugohe3/ppt-master](https://github.com/hugohe3/ppt-master) (style spec repository) | Swiss Grid style and other design specs → stored as the `swiss-grid` style |
| [corazzon/pptx-design-styles](https://github.com/corazzon/pptx-design-styles) / [sunchaokun/PPT-Design-Skill](https://github.com/sunchaokun/PPT-Design-Skill) / [GordenSun/GordenPPTSkill](https://github.com/GordenSun/GordenPPTSkill) | Research on community PPT style systems and template practice (only borrowing style specs, not copying templates) |
| [SlideRabbit 2025 design trends](https://sliderabbit.com/blog/inspiring-2025-presentation-design-trends/) and others | Brutalism / Y2K / gradient trends → stored as `brutalist` / `retro-y2k` / `aurora-gradient` |
| [Storyset](https://storyset.com/) / [unDraw](https://undraw.co/) / [manypixels](https://www.manypixels.co/gallery) | Free flat-illustration ecosystem → `flat-illustration` style and image-generation mapping reference |

All of the above sources passed security and health review (prompt injection / malicious instructions / data exfiltration / copyright licensing / active health, 2026-08-24); only the essence was extracted, adapted, and rewritten into the library, with no verbatim copying; style catalog entries all cite source links.

## V·5 Blueprint Reuse

Existing blueprint: **办公文档** (`blueprints/办公文档.md`, containing the topology diagram / agent list / ADR decision records / community reusable parts / lessons learned). For similar needs (document-type multi-agent workflows), you can say 「**复用 办公文档 拓扑**」 — only clarify differences, no redesign; iterative changes are written back to both the blueprint and this document.

## VI. First-Run Instructions

1. New task: just say 「**帮我做一份[PPT/Word/Excel]**」 or 「**开始策划**」, and orchestrator will ask about topic / audience / length / style, **and ask whether there are brand assets (if so, produce uniformly per the brand kit)**.
2. Choose style: orchestrator first asks for a direction (steady-professional / modern-premium / distinctive-personality / friendly-content), then offers 3–4 specific style options (14 styles: Business Professional / McKinsey / Academic / Minimal / Creative / Official + Swiss Grid / Glassmorphism / Magazine Editorial / Dark Tech / Aurora Gradient / Flat Illustration / Brutalist / Retro Y2K), with community source links for reference.
3. After confirming the plan, say the corresponding expert trigger word (e.g. 「制作PPT」) to produce the document.
4. Modify (an already-delivered document): just say what to change, and orchestrator will use `send_message` to have the atomic agent continue the conversation for revision.
5. **Not satisfied with the style? Switch a round**: just say 「换一种风格/风格不满意/换个风格重做」 — content and structure are fully preserved, orchestrator re-offers style options, produces a new plan with a new `style_id` and redoes it, the new file does not overwrite the old one, and delivery includes a before/after style comparison. Recommend at most 3 consecutive rounds, then switch to detail fine-tuning or provide a reference image.
6. **Edit an existing document**: send me the .docx/.pptx/.xlsx path (or drag it in) and say 「**改这个Word / 改这份PPT / 修改表格** + what to change」, orchestrator first identifies the original document's structure/style, then has the expert make incremental changes, and delivery includes an itemized change list without overwriting the original file.

## VII. Security Gate

**Security gate passed (2026-08-23)**: All AGENT.md files and knowledge/ passed five reviews — prompt injection / malicious instructions / data exfiltration / supply-chain poisoning / platform security; community source content is traceable, with review conclusions attached.
**Security gate re-review passed (2026-08-24, after the v2 brand-kit iteration)**: The newly added `shared/brand-kit.md` and the brand-kit sections of each AGENT.md passed re-review (no prompt injection / malicious instructions / data exfiltration / credential residue), and the trigger-word registry was frozen unchanged.

## VIII. v3 PPT Capability Enhancements (2026, distilled after researching 7 community PPT plugins)

> Borrowing the **mechanisms** (not the code) of dsh-ppt / PPTKit Presentation / @yejiming/dsh-ppt / pptfast / pptwise / DeepSeek Design / dsh-univer-office, slide-designer adds four capabilities:

- **Visual review QA**: `agents/slide-designer/scripts/visual-qa.py` automatically detects text overflow / element overlap / insufficient contrast / out-of-bounds / missing fonts (`python visual-qa.py <deck.pptx>`); `knowledge/visual-qa.md` provides a manual inspection checklist and auto-fix rules.
- **Plugin ecosystem collaboration**: `knowledge/plugin-ecosystem.md` records the install commands and collaboration modes of 7 community PPT plugins (this skill produces the narrative first draft → plugins handle visualization refinement / review).
- **Theme alias mapping**: `shared/style-catalog.md` adds mappings from community theme names to `style_id` (Data Drift→`dark-tech`, Swiss Pulse→`swiss-grid`, Velvet Standard→`editorial-magazine`, etc.).
- **Enhancements**: spec-first (page-level specs finalized before rendering), dual HTML-preview artifacts, extracting colors/fonts from the company's existing PPTs into the brand-kit.
