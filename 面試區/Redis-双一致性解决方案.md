---
界: NoSQL
门: Redis
纲: 
tags: ["#NoSQL/#Redis","#BQ"]
aliases:
  - 
date: 2021-09-07
---

### 決定雙一致性方案所需要提前資訊
- 資料的讀寫頻率
- 資料的重要性

# 一致性解決方案
![[Redis的一致性#Cache-Aside Pattern]]

![[Redis的一致性#Read-Through Write-Through（读写穿透）]]

![[Redis的一致性#Write behind （异步缓存写入）]]