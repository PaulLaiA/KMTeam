---
界: DataBase
門: 
綱: 
tags: ["#Java","#IO","#BQ"]
aliases:
  - 
date: 2021-09-07
---

悲觀鎖：在數據處理過程，將數據處於鎖定狀態，一般使用數據庫的鎖機制實現。（行鎖、表鎖、讀鎖、寫鎖、共享鎖、排他鎖）

-   表鎖
    -   lock table 表名稱 read|write,表名稱2 read|write;//增加表鎖
    -   unlock tables;//刪除表鎖
-   共享鎖（行鎖-讀鎖）
    select ... lock in share mode
-   排他鎖（行鎖-寫鎖）
    SQL末尾加上for update

樂觀鎖：對數據進行校驗和比較，來保證本次更新是最新的，否則就失敗

**注意：排他鎖（寫鎖）使用不當（沒有使用索引）可能會導致整張表被鎖住，儘可能使用樂觀鎖。**

