---
date: 2021-11-04
aliases: []
---

# Metadata

**Title** :: Study_Matthew

**Author** :: #Matthew 

**Status** :: #🌱

**Type** :: #Node

---

```dataview
list Status + " - " + Title from ""
where contains(ParentNode,[[Study_Matthew/Study_Matthew]])
and !contains(file.name,"107")
sort file.name
```

