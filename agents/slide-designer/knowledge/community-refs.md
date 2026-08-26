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

## 2026 社区 PPT 插件生态（调研入库，详情见 plugin-ecosystem.md）

| 来源 | 链接 | 借鉴点 | 审查结论 |
|------|------|--------|----------|
| STARDUSTLC666/dsh-ppt | https://github.com/STARDUSTLC666/dsh-ppt | HTML+PPTX 双产物、5 视觉主题、零依赖 | 通过（仅借鉴机制，未照搬代码） |
| openHacking/PPTKit Presentation | cnblogs 部署指南 | deck-brief + deck-spec 中间文件，可追溯可迭代 | 通过（同上） |
| @yejiming/dsh-ppt | https://www.npmjs.com/package/@yejiming/dsh-ppt | 原生 PPTX + 免版权配图 + 自动图表 + AI 视觉审稿 | 通过（审稿规则提炼入 visual-qa） |
| liustack/pptfast | https://github.com/liustack/pptfast | 抽公司配色字体、可视化批注修改、语义 IR | 通过（配色抽取并入 brand-kit） |
| liustack/pptwise | https://github.com/liustack/pptwise | 语义 IR + 24 风格 + 实时预览迭代 | 通过（IR 并入 spec 先行） |
| iPolloWork/DeepSeek Design | https://github.com/Devin-AXIS/deepseek-design | 可视化工作台 + 选区级 Ask AI | 通过（定位下游协同） |
| dream-num/dsh-univer-office | https://github.com/dream-num/dsh-univer-office | 在线编辑 + 溢出/重叠检测 + 导入 .pptx | 通过（检测规则提炼入 visual-qa） |

> 避坑：以上仅借鉴「机制/规则」，不照搬私有模板与素材；安装第三方插件=在 harness 内运行第三方代码，需自行审查、限最小权限。
