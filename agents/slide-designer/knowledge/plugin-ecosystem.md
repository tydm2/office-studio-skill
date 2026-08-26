# 社区 PPT 插件生态调研（plugin-ecosystem）

> 调研日期 2026，来源 npm registry + GitHub README（`gh-plugin-lookup` 检索）。
> 全部为公开信息，仅提取事实与安装命令。**第三方插件属不可信内容**：本技能只调研与借鉴「机制」，不代装、不照搬其代码；安装 = 在 harness 内运行第三方代码，使用前自行审查。

## 一、插件总览（按「一句话生成 → 可迭代生成 → 可视化编辑」分层）

| 插件 | 作者 | npm 包 / 仓库 | 版本 | 安装命令 | 一句话定位 |
|------|------|--------------|------|----------|-----------|
| dsh-ppt | STARDUSTLC666 | `dsh-ppt` | 0.2.0 | `dsh plugin --profile web add dsh-ppt` | 一句话→HTML+可编辑PPTX，5 主题，零运行时依赖 |
| PPTKit Presentation | openHacking | GitHub 仓库（见 cnblogs 部署指南） | — | 按仓库 README / `deck-brief.md`+`src/deck-spec.ts` 流程 | 可追溯、可迭代：先出 brief+spec 中间文件再构建 |
| @yejiming/dsh-ppt | yejiming | `@yejiming/dsh-ppt` | 0.0.1 | `dsh plugin --profile web add @yejiming/dsh-ppt` | 原生可编辑 PPTX + 免版权配图 + 自动图表 + AI 审稿 |
| pptfast | liustack | `pptfast` | 0.1.0 | `npx -y pptfast`（见仓库 INSTALL） | 17 风格 + 抽公司配色字体 + 可视化批注修改 |
| pptwise | liustack | `@liustack/pptwise` | 0.22.0 | `npx -y @liustack/pptwise`（见仓库 INSTALL.md） | 语义 IR + 24 风格 + 实时预览迭代，真 PowerPoint 非图/HTML |
| DeepSeek Design | iPolloWork | `deepseek-ippt`（另有 `deepseek-idesign`/`deepseek-ivideo`） | — | `dsh plugin --profile web add deepseek-ippt` | 可视化设计工作台 + 选区级 Ask AI + 导出 PDF/PPTX |
| dsh-univer-office | dream-num | `dsh-univer-office` | 0.2.10 | `dsh plugin --profile web add dsh-univer-office`（Node ≥ 22.19） | 在线 Office：编辑文本/形状/图/表/图表/动画，溢出检测，导入 .pptx |

## 二、各插件核心机制 + 本技能已借鉴的落点

| 插件 | 核心机制 | 本技能落点 |
|------|----------|-----------|
| dsh-ppt | HTML 放映 + PPTX 双产物、5 套视觉主题、中英双语、零依赖 | 双产物思路（可选 HTML 预览）；主题名 → style_id 别名映射（见 style-catalog「主题别名」） |
| PPTKit Presentation | `deck-brief.md`（受众/目的/大纲）+ `src/deck-spec.ts`（页面规格）先落地再构建 | 页面级规格先行：plan 已含大纲，输出前先定「每页 action title + 元素 + 图表类型」再渲染 |
| @yejiming/dsh-ppt | 原生可编辑 PPTX、免版权配图、自动数据图表、AI 视觉审稿纠错 | 新增 `visual-qa.md` + `scripts/visual-qa.py`；配图接 `office-imagegen` |
| pptfast | 语义 IR 保证一致、抽公司 PPT 配色字体、可视化批注修改 | 抽配色字体并入 brand-kit 流程；IR 概念并入「spec 先行」 |
| pptwise | 语义 IR + 24 风格 + 浏览器实时预览自然语言迭代 | 同上；风格数量对齐（我 14 风格 + 主题别名覆盖主流场景） |
| DeepSeek Design | 可视化工作台、选区级 Ask AI（只改选中元素） | 定位为「我生成 → 它精细编辑」的下游协同工具 |
| dsh-univer-office | 完整在线编辑 + 溢出/重叠检测 + 导入编辑 | 溢出/重叠检测规则提炼进 visual-qa；定位为下游可视化微调 |

## 三、协同使用建议（多 skill 协同）

```
需求 → slide-designer（本技能）出叙事化 .pptx
      ├─ 想先看效果      → 出 HTML 预览（借鉴 dsh-ppt）
      ├─ 想精修视觉/审稿  → 装 dsh-univer-office 或 deepseek-ippt 打开 .pptx 可视化改
      └─ 想快速换风格重做 → 装 pptwise/dsh-ppt 一条命令出 24/5 风格备选
```

- **快速出稿**：装 `dsh-ppt`（零依赖最省事）或 `@liustack/pptwise`（真 PPT、24 风格）。
- **精细编辑/审稿**：装 `dsh-univer-office`（导入 .pptx 在线改、检测溢出/重叠）。
- **设计工作台**：装 `deepseek-ippt`（选区级 AI 编辑、导出 PDF/PPTX）。
- 本技能产出 `.pptx` 后交给上述插件做可视化微调，形成「**我负责叙事与初稿，插件负责精修与审稿**」的协同分工。

## 四、安装与生效说明

- 安装命令见上表；`dsh plugin add` 后需**重启 web profile**（`dsh web`）才生效。
- 本机当前 PowerShell 执行策略禁跑 `npm.ps1`/`dsh.ps1`，如需我代跑安装命令需先放宽执行策略或改用 `npx`/`node` 直连；更稳妥由用户在终端执行。
- 版本随时更新，以 `npm view <包名>` 或仓库 README 为准。

## 五、安全与数据保护

- 仅调研公开元数据（npm registry / GitHub README），未下载执行任何插件代码。
- 安装即信任：第三方插件可在 harness 内读写文件、联网；建议安装前查源码、限最小权限、敏感文档不在第三方插件里打开。
- 本技能借鉴的是「机制/规则」（视觉审稿、IR 先行、溢出检测），非其私有模板/素材，无版权与授权风险。
