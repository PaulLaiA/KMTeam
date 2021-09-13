---
界: DataBase
門: SQL
綱: MySQL
tags: ["#DB","#MySQL","#BQ"]
aliases:
  - 
date: 2021-09-07
---

-   hash
    
    Hash底層實現是由Hash表來實現的，是根據鍵值 <key,value> 儲存數據的結構。非常適合根據key查詢value值，也就是單個key查詢，或者說等值查詢。
    
-   B+Tree
    
    -   非葉子節點不儲存data數據，只儲存索引值，這樣便於儲存更多的索引值
    -   葉子節點包含了所有的索引值和data數據
    -   葉子節點用指針連線，提高區間的訪問效能