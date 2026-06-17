# 📚 文献索引

> 自动汇总 `literature/` 下所有文献笔记（dataview 生成，无需手动维护）。
> 看不到表格？请确认已启用 dataview 插件。

```dataview
TABLE
  year AS "年份",
  venue AS "来源",
  status AS "状态",
  contributor AS "贡献者",
  scope AS "适用范围"
FROM "literature"
WHERE file.name != "index"
SORT year DESC
```

## 按贡献者统计
```dataview
TABLE length(rows) AS "笔记数"
FROM "literature"
GROUP BY contributor AS "贡献者"
SORT length(rows) DESC
```
