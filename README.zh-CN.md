# office-studio —— 办公文档多智能体工作流

[English](README.md) · **[简体中文](README.zh-CN.md)** · [Español](README.es.md) · [Français](README.fr.md) · [Русский](README.ru.md) · [日本語](README.ja.md)


> 目标平台：**DSH**（本环境）。一条流水线高效制作/编辑 **Word · PPT · Excel** 三类办公文档，成果质量优先，支持**多种风格一键切换**。

## 一、流水线图

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

## 二、触发词映射表

| 智能体 | 一句话职责 | 触发词（自然语言，含同义变体） |
|--------|-----------|------------------------------|
| **orchestrator**（大脑） | 吃需求 → 吐已选风格+自包含任务包 → 汇总交付 | 新建：`开始策划` `新建文档` `帮我做一份PPT` `帮我写一份Word` `帮我做一份Excel` `做文档`；编辑：`编辑文档` `修改文档` `改这份PPT` `改这个Word`（用户提供现有文件时优先走编辑分支） |
| **word-crafter** | 吃 plan+风格 → 吐规范 .docx | 新建：`撰写Word` `写正文` `生成Word`；编辑：`改Word` `修改文档` `编辑这份Word` |
| **slide-designer** | 吃 plan+风格 → 吐叙事化 .pptx | 新建：`制作PPT` `做幻灯片` `生成PPT`；编辑：`改PPT` `修改这份PPT` |
| **sheet-analyst** | 吃数据+plan+风格 → 吐结论先行 .xlsx | 新建：`制作Excel` `做报表` `生成Excel` `分析数据`；编辑：`改Excel` `修改表格` `编辑这份Excel` |

**触发规则**：新任务（无对应 plan 存档）一律先进 `orchestrator` 澄清；`plan-NN.md` 经用户确认后，用各专家专属触发词直达。触发词互不冲突，也不与系统内置命令冲突。

## 三、目录结构

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

## 四、文件契约（上游产出 → 下游读取）

1. **策划 → 各专家（新建）**：`outputs/plans/plan-NN.md`，头部元信息格式固定：
   ```yaml
   ---
   doc_id: plan-NN
   title: <标题>
   doc_type: word | ppt | excel
   style_id: <见 shared/style-catalog.md，14 种：6 经典 + 8 社区借鉴；选择器见 shared/style-switcher.md>
   brand_kit: <可选，用户提供品牌资产时写 brand-NN，规则见 shared/brand-kit.md；无则省略>
   audience: <受众>
   purpose: <用途>
   length: <篇幅>
   data_source: <可选，Excel 数据文件路径>
   ---
   ```
1b. **编辑任务 → 各专家（编辑）**：`outputs/plans/edit-NN.md`：
   ```yaml
   ---
   doc_id: edit-NN
   source_file: <原文档路径，用户拖入/提供>
   doc_type: word | ppt | excel
   mode: edit
   change_request: <修改要求，具体到段落/页面/单元格>
   keep: <必须保留项：原格式/已有结论/数据>
   style_id: <默认沿用原风格；换风格需用户明确指定>
   ---
   ```
2. **各专家产出**：`outputs/<type>/<文件名>`，并附带同名 `-meta.md`（记录：doc_id、style_id、**brand_kit（若有）**、生成日期、数据来源、改动说明）。
3. **迭代**：用户修改意见一律通过 DSH `send_message` 给原子代理续聊修订，保留上下文；修订必须写进 `-meta.md` 的「改动说明」，不静默重写已确认内容。
4. **品牌套件（v2）**：用户提供品牌资产（品牌色/Logo/字体/模板）→ orchestrator 收集进 `shared/brand-kit.md`，plan 写 `brand_kit: brand-NN`，三专家统一套用（品牌优先于风格库）；同一 brand_id 多件套交付时 orchestrator 核对品牌一致性（见 `shared/brand-kit.md`）。

## 五、社区先例致谢（设计参考）

| 来源 | 借鉴点 |
|------|--------|
| [anthropics/skills](https://github.com/anthropics/skills)（官方） | docx/pptx/xlsx「先读现有文件→生成→校验可打开」范式 |
| [johnson7788/skill-ppt-agents](https://github.com/johnson7788/skill-ppt-agents) + [MultiAgentPPT](https://github.com/johnson7788/MultiAgentPPT) | PPT「大纲→内容→设计」多阶段、每阶段独立验收 |
| [rafalozan0/DocFlow](https://github.com/rafalozan0/DocFlow-Presentations-and-Docs-Skill) | 三件套统一 python-pptx/docx/openpyxl + 文件化模板目录（多风格切换） |
| [cabbage2000-lab/data-analysis-skills](https://github.com/cabbage2000-lab/data-analysis-skills) | Excel 结论先行六段式、数据可追溯、不杜撰、相关≠因果 |
| [fleurytian/awesome-claude-skills](https://github.com/fleurytian/awesome-claude-skills)（前麦肯锡） | 金字塔原理 / MECE 叙事结构 |
| [hugohe3/ppt-master](https://github.com/hugohe3/ppt-master)（风格规格仓库） | 瑞士网格风等设计规格 → 入库为 `swiss-grid` 风格 |
| [corazzon/pptx-design-styles](https://github.com/corazzon/pptx-design-styles) / [sunchaokun/PPT-Design-Skill](https://github.com/sunchaokun/PPT-Design-Skill) / [GordenSun/GordenPPTSkill](https://github.com/GordenSun/GordenPPTSkill) | 社区 PPT 风格体系与模板实践调研（仅借鉴风格规格，不搬模板） |
| [SlideRabbit 2025 设计趋势](https://sliderabbit.com/blog/inspiring-2025-presentation-design-trends/) 等 | 粗野主义/Y2K/渐变等趋势 → 入库为 `brutalist` / `retro-y2k` / `aurora-gradient` |
| [Storyset](https://storyset.com/) / [unDraw](https://undraw.co/) / [manypixels](https://www.manypixels.co/gallery) | 免费扁平插画生态 → `flat-illustration` 风格与生图映射参考 |

以上来源均通过安全健康审查（提示注入/恶意指令/数据外泄/版权许可/活跃健康，2026-08-24），仅提炼精华、适配改写后入库，未整段照搬；风格库条目均注明来源链接。

## 五·五、Blueprint 复用
已有 blueprint：**办公文档**（`blueprints/办公文档.md`，含拓扑图/智能体清单/ADR 决策记录/社区可复用零件/避坑经验）。同类需求（文档类多智能体工作流）可说「**复用 办公文档 拓扑**」——只澄清差异，不重新设计；迭代改动同步回写 blueprint 与本文档。

## 六、首跑指令

1. 新建任务：直接说「**帮我做一份[PPT/Word/Excel]**」或「**开始策划**」，orchestrator 会问清主题/受众/篇幅/风格，**并询问是否有品牌资产（有则按品牌套件统一出品）**。
2. 选风格：orchestrator 先问方向（稳重专业/现代高级/个性鲜明/亲和内容），再给 3-4 个具体风格选项（14 种风格：商务专业/麦肯锡/学术/极简/创意/公文 + 瑞士网格/玻璃拟态/杂志编辑/暗色科技/极光渐变/扁平插画/粗野主义/复古千禧），可参考社区来源链接。
3. 确认 plan 后，说对应专家触发词（如「制作PPT」）即可产出文档。
4. 修改（已交付文档）：直接说改哪里，orchestrator 会用 `send_message` 让原子代理续聊修订。
5. **风格不满意？换一轮**：直接说「换一种风格/风格不满意/换个风格重做」——内容结构全部保留，orchestrator 重新给风格选项，出新 plan 换 `style_id` 重做，新文件不覆盖旧文件，交付时附新旧风格对比。建议最多连续 3 轮，之后转细节微调或提供参考图。
6. **编辑现有文档**：把 .docx/.pptx/.xlsx 路径发给我（或拖入），说「**改这个Word / 改这份PPT / 修改表格** + 要改什么」，orchestrator 会先识别原文档结构/风格，再让专家增量修改，交付时附逐条改动清单，原文件不覆盖。

## 七、安全门禁

**安全门禁通过（2026-08-23）**：全部 AGENT.md 与 knowledge/ 已通过提示注入 / 恶意指令 / 数据外泄 / 供应链投毒 / 平台安全五项审查，社区来源内容可追溯、审查结论随附。
**安全门禁复查通过（2026-08-24，v2 品牌套件迭代后）**：新增 `shared/brand-kit.md` 与各 AGENT.md 品牌套件段落复查通过（无提示注入/恶意指令/数据外泄/密文残留），触发词登记表冻结未动。

## 八、v3 PPT 能力增强（2026，调研 7 个社区 PPT 插件后提炼）

> 借鉴 dsh-ppt / PPTKit Presentation / @yejiming/dsh-ppt / pptfast / pptwise / DeepSeek Design / dsh-univer-office 的**机制**（非代码），slide-designer 新增四项能力：

- **视觉审稿质检**：`agents/slide-designer/scripts/visual-qa.py` 自动检测文字溢出 / 元素重叠 / 对比度不足 / 越界 / 字体缺失（`python visual-qa.py <deck.pptx>`）；`knowledge/visual-qa.md` 提供人工目检清单与自动修正规则。
- **插件生态协同**：`knowledge/plugin-ecosystem.md` 记录 7 个社区 PPT 插件的安装命令与协同方式（本技能出叙事化初稿 → 插件做可视化精修/审稿）。
- **主题别名映射**：`shared/style-catalog.md` 增加社区主题名 → `style_id` 映射（数据漂移→`dark-tech`、瑞士脉冲→`swiss-grid`、天鹅绒标准→`editorial-magazine` 等）。
- **增强项**：spec 先行（渲染前落页面级规格）、HTML 预览双产物、从公司现有 PPT 抽取配色/字体并入 brand-kit。
