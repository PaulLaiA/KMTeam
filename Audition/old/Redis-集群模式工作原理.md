---
界: NoSQL
門: Redis
綱: 
tags: ["#NoSQL","#Redis","#BQ"]
aliases:
  - 
date: 2021-09-07
---

# 集中式
    
# Gossip
gossip協議包含多種訊息：ping、pong、meet、fail

-   meet：某個節點發送 meet 給新加⼊的節點，讓新節點加⼊集群中，然後新節點就會開始與 其它節點進⾏通訊。
-   每個節點都會頻繁給其它節點發送 ping，其中包含⾃⼰的狀態還有⾃⼰維護的集群 後設資料，互相通過 ping 交換後設資料。
-   返回 ping 和 meeet，包含⾃⼰的狀態和其它資訊，也⽤于資訊⼴播和更新。
-   某個節點判斷另⼀個節點 fail 之後，就發送 fail 給其它節點，通知其它節點說，某個節 點宕機啦。