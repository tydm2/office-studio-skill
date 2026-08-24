# Word 输出模板（output-template）

## 文件产出
1. `outputs/word/doc-NN.docx`（NN 递增，先读存档防重复）
2. `outputs/word/doc-NN-meta.md`：
   ```yaml
   ---
   doc_id: doc-NN
   plan_id: plan-NN
   style_id: <style_id>
   generated: <YYYY-MM-DD>
   data_source: <可选>
   changes: <本次改动说明>
   ---
   ```

## docx 结构骨架（按文档类型微调）
```
[封面（可选，商务/学术必选）]
  标题 / 副标题 / 单位 / 日期
[目录（长文档可选）]
[正文]
  一、结论（开篇结论）
  二、主体（按范式分节）
  三、下一步/建议
[图表]（每图配解读句）
[参考文献/附录（学术/公文必选）]
```

## 一段优质范例（好在：结论先行 + 数据具体 + 无套话）
> 三季度营收 1.24 亿元，环比增长 12%，主要来自华东区新客户签约 31 家。增长集中在 B2B 业务线，客单价同比提升 8%，但华南区流失率升至 6.5%，需在下季度重点跟进。
