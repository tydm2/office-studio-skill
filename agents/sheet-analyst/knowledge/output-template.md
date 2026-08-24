# Excel 工作表结构模板（output-template）

## 文件产出
1. `outputs/excel/sheet-NN.xlsx`
2. `outputs/excel/sheet-NN-meta.md`：
   ```yaml
   ---
   sheet_id: sheet-NN
   plan_id: plan-NN
   style_id: <style_id>
   generated: <YYYY-MM-DD>
   data_source: <源数据路径>
   caliber: <口径说明：周期/单位/计算方式>
   changes: <本次改动说明>
   ---
   ```

## 工作表结构（sheet 顺序即阅读顺序）
```
① 结论（置顶）：每条=结论句 + 支撑数据 + 指向工作表
② 原始数据：不动，保证可追溯
③ 清洗后数据：处理过程注明
④ 计算指标：公式与说明单元格
⑤ 图表：每图配解读句
```

## 结论页范例（好在：结论先行 + 数据可指回 + 不越界推断）
> 1. Q3 营收 1.24 亿元，环比 +12%，主要由华东区贡献（见"计算指标"表 E 列）。
> 2. 华南区流失率升至 6.5%，为近四季度最高（见"原始数据"表 B 列）；具体原因需结合客户访谈验证，暂无数据支撑因果结论。
