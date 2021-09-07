# 今日新增
```dataview  
table 分类,创建日期,file.etags as 标签
from "个人区/面试区/面试题/Java"
WHERE file.cday > date(today) - dur(1 day) 
and !contains(file.name,"大纲")
sort file.ctime
```

---
# 今日修正
```dataview  
table 分类,创建日期,file.etags as 标签
from "个人区/面试区/面试题/Java"
WHERE file.mday > date(today) - dur(1 day) 
and !contains(file.name,"大纲")
and file.cday!=file.mday
sort file.ctime
```

---
# 笔记大纲
```dataview
table 分类,创建日期,file.etags as 标签
from "个人区/面试区/面试题/Java"
where !contains(file.name,"大纲")
sort file.ctime
```