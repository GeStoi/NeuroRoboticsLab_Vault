# 🔤 术语表

> 自动汇总 `glossary/` 下所有术语（dataview 生成）。一术语一文件，新建即自动收录。

```dataview
TABLE
  term_zh AS "中文",
  term_en AS "English"
FROM "glossary"
SORT file.name ASC
```

---

# ✏️ 符号表

```dataview
TABLE file.link AS "符号条目"
FROM "notation"
SORT file.name ASC
```

# 🧪 方法卡片

```dataview
TABLE file.link AS "方法"
FROM "methods"
SORT file.name ASC
```
