---
date: { { DATE:YYYY-MM-DD } }
aliases: []
---

# Metadata

**Title** :: Go 面试题

**Author** :: #dPaulLai

**Status** :: #🌱

**Type** :: #View

**ParentNode** :: [[View/View]]

---

# 基础题

```dataview
list 	Status + " - | " + Classification + " - |  [[Study_dPaul/Audition/Golang/" + file.name + "|" + Question + "]]"
from 	"Study_dPaul"
where 	contains(ParentNode , [[Study_dPaul/Audition/Golang/Golang]])
and		!contains(file.name,"Golang")
and		contains(Classification,"#Basic")
sort file.name
```

# 并发编程

```dataview
list 	Status + " - | " + Classification + " - |  [[Study_dPaul/Audition/Golang/" + file.name + "|" + Question + "]]"
from 	"Study_dPaul"
where 	contains(ParentNode , [[Study_dPaul/Audition/Golang/Golang]])
and		!contains(file.name,"Golang")
and		contains(Classification,"#Concurrent")
sort file.name
```