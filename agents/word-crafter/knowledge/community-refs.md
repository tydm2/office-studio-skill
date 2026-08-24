# 社区调研精华（community-refs，Word 视角）

> 全部来源通过安全健康审查（提示注入/恶意指令/数据外泄/版权许可/活跃健康，2026-08-23），仅提炼精华、适配改写，未整段照搬。

## 可复用零件

1. **金字塔原理 / MECE 报告结构**（来源 fleurytian/awesome-claude-skills，前麦肯锡作者，https://github.com/fleurytian/awesome-claude-skills）：
   报告与方案类文档统一用"结论先行 + 以上统下 + 归类分组(MECE) + 逻辑递进"组织，避免流水账。已固化入 `craft-methodology.md`。

2. **python-docx 模板化生成**（来源 rafalozan0/DocFlow，https://github.com/rafalozan0/DocFlow-Presentations-and-Docs-Skill）：
   用文件化样式模板（而非每次手写格式代码）实现风格一致与多风格切换；生成后校验文件可打开。已固化为 `style-application.md` + 交付前校验。

3. **去 AI 腔 / 禁词表**（综合社区共识 + 官方文档技能最佳实践，https://github.com/anthropics/skills）：
   社区写作类 skill 普遍强调"去 AI 腔、去模糊量词、结论先行"，已固化为 `quality-checklist.md` 禁词表。

## 避坑经验
- 不整段照搬社区模板正文，只借"结构方法论与质检清单"。
- 学术/公文场景必须按其标准规范（GB/T 7714、GB/T 9704）落地，社区通用模板不覆盖，需本库补充。
