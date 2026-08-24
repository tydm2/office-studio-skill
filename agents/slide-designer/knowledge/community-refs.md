# 社区调研精华（community-refs，PPT 视角）

> 全部来源通过安全健康审查（提示注入/恶意指令/数据外泄/版权许可/活跃健康，2026-08-23），仅提炼精华、适配改写，未整段照搬。

## 可复用零件

1. **PPT 多阶段流水线**（来源 johnson7788/skill-ppt-agents + MultiAgentPPT，https://github.com/johnson7788/skill-ppt-agents）：
   把 PPT 拆「大纲→内容→设计」多阶段，每阶段产出可独立验收，问题早发现。本工作流对应：orchestrator 出大纲/plan → 小组叙事专家出内容 → 视觉专家落地设计。

2. **action title + 每页一论点 + 金融图表可视化**（来源 fleurytian/awesome-claude-skills，前麦肯锡，https://github.com/fleurytian/awesome-claude-skills）：
   标题必须是结论、每页只证明一个 message、图表为结论服务。已固化入 `narrative-methodology.md`。

3. **pptx 创建/编辑校验范式**（来源 anthropics/skills 官方，https://github.com/anthropics/skills）：
   编辑现有 pptx 时先读结构再增量改，交付前校验可打开。已固化为交付红线。

## 避坑经验
- 社区 PPT skill 多绑定私有模板/脚本，只借"叙事结构与质检清单"，不借脚本与模板文件。
- "每页一个论点 + 信息密度受控"是提升 PPT 质量最有效的一条，比任何花哨版式都重要。
