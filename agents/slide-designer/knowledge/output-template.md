# PPT 页面骨架模板（output-template）

## 文件产出
1. `outputs/ppt/deck-NN.pptx`
2. `outputs/ppt/deck-NN-meta.md`：
   ```yaml
   ---
   deck_id: deck-NN
   plan_id: plan-NN
   style_id: <style_id>
   generated: <YYYY-MM-DD>
   changes: <本次改动说明>
   ---
   ```

## 页面骨架（按叙事弧微调）
```
P1  封面：主标题（结论化）+ 副标题 + 汇报人/日期
P2  目录/议程
P3+ 正文：每页 action title + 单图/单点支撑
…   （问题→方案→论证→结果）
倒数第2页：结论/行动呼吁
末页：致谢 + 联系方式/落款
```

## 一页优质范例（好在：标题即结论 + 单图 + 解读句）
> **标题：华东区是下季度唯一增长引擎**
> （柱状图：各区营收环比）→ 解读句："华东环比 +18%，其余四区均低于 5%，资源应向华东倾斜。"
