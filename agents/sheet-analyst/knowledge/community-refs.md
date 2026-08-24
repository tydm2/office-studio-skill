# 社区调研精华（community-refs，Excel 视角）

> 全部来源通过安全健康审查（提示注入/恶意指令/数据外泄/版权许可/活跃健康，2026-08-23），仅提炼精华、适配改写，未整段照搬。

## 可复用零件

1. **结论先行六段式 + 每图配解读 + 数据可追溯**（来源 cabbage2000-lab/data-analysis-skills，https://github.com/cabbage2000-lab/data-analysis-skills）：
   数据文件→分析报告的完整范式：先确认行业与受众→清洗→分析→结论先行的六段式报告，每张图配解读，处理过程可追溯。已固化入 `analysis-methodology.md`。

2. **不杜撰 / 相关≠因果 / 小样本不下结论**（来源同上）：
   三条统计铁律是本智能体质量红线核心，社区实践验证其必要性（避免 AI 幻觉数据结论）。

3. **openpyxl 模板化生成与交付校验**（来源 rafalozan0/DocFlow，https://github.com/rafalozan0/DocFlow-Presentations-and-Docs-Skill）：
   用样式模板实现风格一致，生成后校验文件可打开、公式可算。已固化为交付红线。

## 避坑经验
- 数据分析最大的坑是"AI 脑补数据/行业基准"，必须用红线锁死。
- 保留原始工作表不动是"可追溯"的最低成本实现。
