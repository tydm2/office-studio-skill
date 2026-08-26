# 品牌套件（brand-kit）—— 跨文档品牌统一（v2，2026-08-24）

> 名字「office-studio（工作室）」的含义：工作室出品应**成套统一**——同一客户的 Word + PPT + Excel 三件套，即使由不同专家产出，也必须共用同一套品牌语言，看起来像一个工作室做的。
> 品牌套件是**项目级**资产，优先于风格库：**有品牌资产 → 以品牌为准，风格库兜底**；无品牌资产 → 按 `style-catalog.md` 的 style_id 执行（现状不变）。

## 一、何时启用

- **orchestrator 澄清时必问**：「是否有品牌资产（品牌色 / Logo / 字体 / 模板 / 页眉页脚）？」用户提供任意一项 → 启用品牌套件。
- 用户没提但文档属于同一客户/同一项目、且可能多件套交付（如「季度经营汇报」要 Word 报告 + PPT 演示 + Excel 数据）→ 主动询问是否统一品牌。
- 启用后，`plan-NN.md` / `edit-NN.md` 元信息写入 `brand_kit: <id>`，三专家统一读取本文件。

## 二、品牌资产清单（澄清时逐项收集，缺项留空）

| 字段 | 说明 | 示例 |
|------|------|------|
| `brand_id` | 品牌套件编号（如 brand-01），写入 plan 元信息 `brand_kit` | brand-01 |
| `brand_name` | 品牌/客户名称 | 某科技集团 |
| `primary_color` | 主色（HEX） | `#0F4C81` |
| `secondary_color` | 辅色（HEX） | `#F2A900` |
| `accent_color` | 强调色（HEX，点缀用） | `#00A0B0` |
| `font_cn` | 中文字体（标题/正文） | 思源黑体 / 微软雅黑 |
| `font_en` | 英文字体 | Inter / Arial |
| `logo` | Logo 路径或占位说明（无则写「文字标识：品牌名+主色」） | outputs/assets/logo.png 或占位 |
| `header_footer` | 页眉页脚规范（文字/页码/版权行） | 页眉左「某集团」右页码，页脚「机密」 |
| `template_ref` | 参考模板文件路径（可选） | outputs/assets/brand-template.docx |
| `tone` | 语气基调（正式/亲和/专业等，与风格语气对齐） | 专业、结论先行 |

## 三、三端统一应用规则（各专家强制）

| 端 | 品牌套件落地要点 |
|----|------------------|
| **Word（word-crafter）** | 标题/正文颜色取自品牌色板；标题字体 font_cn/font_en；页眉页脚按 header_footer；封面放 Logo（有则嵌，无则文字标识）；表格/图表用品牌色 |
| **PPT（slide-designer）** | 母版背景与标题栏用品牌色；强调色点缀关键数据；Logo 放封面与页脚；字体统一；图表配色取自品牌色板 |
| **Excel（sheet-analyst）** | 表头底色用 primary_color、白字；隔行用 secondary_color 淡色调；关键指标高亮用 accent_color；结论页顶部品牌标识 |

**一致性铁律**：
1. 同一次任务（同一 brand_id）的多件套文档，主色/辅色/字体/页眉页脚**逐项一致**，不得各自发挥。
2. 品牌色优先；风格库中的配色仅作**无品牌资产时**的兜底，不可覆盖品牌色。
3. 交付前自检：本文件是否用到了 brand_id 对应的全部已提供资产；缺项（如无 Logo）明确标注占位处理，不臆造。
4. `-meta.md` 记录 `brand_kit: <brand_id>` 与使用到的资产项，便于多件套核对。

## 四、多件套联动（同一品牌三件套）

- 用户要求「同一项目 Word + PPT + Excel」时，orchestrator 出**同一 brand_id** 的多份 plan（或一份组合 plan），提示各专家：**先读本文件，再读共享产出**（如 PPT 引用 Word 的数据结论、Excel 提供 PPT 的图表数据）。
- 交付时 orchestrator 对照 `-meta.md` 的 brand_kit 字段，核对三件套品牌一致性后交付。
