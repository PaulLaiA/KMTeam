---
date: 2021-10-22
aliases: []
---

# Metadata

**Title** :: 查看 Template 作用

**Author** :: #dPaulLai

**Status** :: #🌱

**Topics** :: [[Template/Template]]

**Type** :: #View

**ParentNode** :: [[View/View]]

---

```dataview
table Title,Author,Classification,Status,Type,Topics,Previous,ParentNode
from "Template"
where !contains(file.name,"Template")
sort file.name
```
