---
界: DataBase
门: SQL
纲: MySQL
tags: ["#DataBase/#MySQL","#BQ"]
aliases:
  - 
date: 2021-09-07
---

-   hash
    
    Hash底层实现是由Hash表来实现的，是根据键值 <key,value> 存储数据的结构。非常适合根据key查找value值，也就是单个key查询，或者说等值查询。
    
-   B+Tree
    
    -   非叶子节点不存储data数据，只存储索引值，这样便于存储更多的索引值
    -   叶子节点包含了所有的索引值和data数据
    -   叶子节点用指针连接，提高区间的访问性能