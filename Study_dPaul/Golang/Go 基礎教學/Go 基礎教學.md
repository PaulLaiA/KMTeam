---
Date: 2021-10-22
aliases: []
---

# Metadata

**Title** :: Go 基礎教學

**Author** :: #dPaulLai

**Status** :: #🌞

**Type** :: #Node

**ParentNode** :: [[Golang]]

---

**最近更新**

```dataview
list 	Status+" - "+Title
from 	"Study_dPaul"
where 	contains(ParentNode , [[Go 基礎教學]])
and		(date(file.mtime)-date(today)) < 1
and 	(date(file.ctime)-date(today)) > 7
sort	file.name
```

**本周新建**

```dataview
list 	Status+" - "+Title
from 	"Study_dPaul"
where 	contains(ParentNode , [[Go 基礎教學]])
and		(date(file.ctime)-date(today)) < 7
sort	file.name
```

![[View - Go - 基礎教學#Go - 基礎教學]]
