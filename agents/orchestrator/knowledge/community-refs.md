# 社区调研精华（community-refs，大脑视角）

> 全部来源通过安全健康审查（提示注入/恶意指令/数据外泄/版权许可/活跃健康，2026-08-23），仅提炼精华、适配改写，未整段照搬。

## 可复用零件

1. **"读→生成→校验可打开"范式**（来源 anthropics/skills 官方，https://github.com/anthropics/skills）：
   编辑/生成文档前三步：先读现有文件结构与样式 → 在其上增量生成 → 交付前校验文件可打开且版式无损。适合"编辑现有文档"场景。

2. **三件套统一工具链 + 文件化模板目录**（来源 rafalozan0/DocFlow，https://github.com/rafalozan0/DocFlow-Presentations-and-Docs-Skill）：
   Word/PPT/Excel 统一用 python-docx / python-pptx / openpyxl；"多风格"靠**文件化模板目录**实现（每种风格一套可复用的样式/版式模板，切换风格=换模板）。这正是本工作流 `shared/style-catalog.md` 的设计依据。

3. **多阶段每阶段独立验收**（来源 johnson7788/MultiAgentPPT，https://github.com/johnson7788/MultiAgentPPT）：
   PPT 拆"大纲→内容→设计"三阶段，每阶段产出可独立验收，问题早发现、不带到下游。

4. **结论先行 + 数据可追溯 + 不杜撰**（来源 cabbage2000-lab/data-analysis-skills，https://github.com/cabbage2000-lab/data-analysis-skills）：
   数据文档三大铁律：结论先行六段式、每图配解读、数据处理过程可追溯；不杜撰行业基准、相关≠因果、小样本不硬下结论。已固化为本流水线全局红线。

## 避坑经验
- 不要整段照搬社区 skill 原文进 prompt，须按用户场景适配改写。
- 社区 skill 常绑定私有脚本/路径，只借"方法论与质检清单"，不借脚本。
- 检索到一年以上失活仓库降级，只作思路参考。
