# 省 Token 记忆协议（memory-lean-protocol，v1.7）

> 把 [dsh-memory-toolkit](https://github.com/123caiji/dsh-memory-toolkit)（123caiji，MIT）的 **lean-context 五层省 Token 编排** + **graph-memory 知识图谱记忆**，提炼为 office-studio `feedback-log` / `usage-log` / 知识库的轻量协议。
> **只借鉴机制，不搬代码、不装插件**；来源为公开 README（2026 检索），机制适配改写。上游参考：graph-memory 衍生自 adoresever/graph-memory；lean-context 纪律层参考 phoenixlucky/zerotoken-skill 的 ZeroToken 六模式。

## 一、纪律（discipline）：记忆写入纪律

- 日志条目只写**结构化短行**：`[日期] 对象 | 意图 | 需求本质(≤120字) | 期望(可验收) | 上下文要点(1-3条) | 优先级`，绝不写散文/过程描述。
- 默认 **ZeroToken 精简输出**：迭代汇报只给「改了什么 / 验证结果 / 遗留」三行；用户要详细才展开（说「详细解释」退出精简）。
- **蒸馏优于堆叠**：重复需求更新原条目，不新增堆积。

## 二、收敛（convergence）：格式与引用收敛

- **字段最小化**：feedback 条目 6 字段封顶；usage 条目 4 类封顶（TRIGGER OK / TRIGGER MISS / LOAD FAIL / EXEC POOR）。
- **引用按「文件名#锚点」不粘全文**：如 `shared/style-catalog.md#主题别名映射`，不复制整段。
- **长内容拆 references/ 与 archive/**，正文只留索引行。

## 三、路由（routing）：按需加载（0 注入）

- `feedback-log` / `usage-log` / 知识库**平时不读取（0 token 注入）**。
- 仅「迭代 / 体检 / 审计」时全量读取（用户说「优化/迭代 xx」才读）。
- 知识库分级：正文 ≤100 行精简，细节进 `references/`，禁止一次性全读。

## 四、审计（audit）：迭代前上下文成本审计

- **轻量估算**：UTF-8 字节 ≈ token 数（中文约 1 字 ≈ 1–1.5 token，快速估算 `字符数 × 1.5` 或 `字节 ÷ 2`）。
- **每次迭代前扫**：本 AGENT.md + knowledge/ + logs 总 token；目标**指令链 ≤ 8k token**（参考：纪律层注入 911 token 换来每会话输出省 70–88%，100 轮 ROI 17x 的成本结构）。
- 检出**重复段落 / 冗余注释 → 收敛**（合并、删、移 archive），不带着冗余迭代。

## 五、重建（rebuild）：记忆重建与归档

- `feedback-log` 活跃需求 **> ~50 条 → 归档**到 `feedback-log.archive.md`（已消费全量移入，活跃区留 ≤20 条）。
- **5-Why 复盘结论写回 blueprint / AGENT.md**，不在 usage-log 反复堆叠过程。
- **轻量图谱化**（graph-memory 的 markdown 版，无需 DB）：
  - 节点类型：`[TASK]`（任务）/ `[SKILL]`（技能·机制）/ `[EVENT]`（事件·决策）
  - 关系标注：`USED_SKILL`（用了哪个机制）/ `SOLVED_BY`（被谁解决）/ `REQUIRES`（依赖）/ `PATCHES`（修补了谁）/ `CONFLICTS_WITH`（与谁冲突）
  - 写法：feedback 条目后追加 `关系:<类型>→<对象>`；blueprint 维护「节点-关系」索引表，双向可查
  - 示例：`[2026-08-26] 对象:slide-designer | 意图:revise | 需求本质:PPT 视觉审稿质检 … | 关系:USED_SKILL→visual-qa, SOLVED_BY→调研 7 插件`

## 六、验证与 ROI

- 每次迭代结束用审计公式回测：日志/知识库 token 是否收敛、无重复段落。
- 参照（dsh-memory-toolkit 实测）：纪律层一次注入 ~911 token，ZeroToken 模式每会话输出省 70–88%（213→27 / 214→65 tokens），100 轮净省 ~15.9k token。

## 安全说明

- 本协议仅借鉴公开机制，不引入任何第三方依赖；日志不含密钥/个人数据；归档文件同样遵守脱敏与最小收集。
