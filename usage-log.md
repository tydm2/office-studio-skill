# office-studio 工作流使用留痕（usage-log，工作流级）

> v1.6 机制：追加式记录四类——`TRIGGER OK`（触发正常）/ `TRIGGER MISS`（触发漏判，记疑似原因）/ `LOAD FAIL`（加载失败）/ `EXEC POOR`（执行质量差，记问题与用户反应）。TRIGGER MISS 与 EXEC POOR 强制升级为 5-Why 复盘（现象→根因≥2层→改进项）。
> v1.7（省 token）：条目默认一行精简、不写过程；5-Why 结论沉淀进 blueprint/AGENT.md，不在本文件堆叠；审计与归档见 `shared/memory-lean-protocol.md`。

- [2026-08-24] ITERATE: 用户要求按名字定向优化 → 落地「品牌套件 v2」（brand-kit.md + orchestrator/三专家/README 同步），触发词登记表冻结未动。
