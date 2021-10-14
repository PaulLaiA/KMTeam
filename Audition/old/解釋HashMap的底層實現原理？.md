---
界: Java
門: Container
綱: 
tags: ["#Java","#Container","#BQ"]
aliases:
  - 
date: 2021-09-12
---



Java中的HashMap是以鍵值對(key-value)的形式儲存元素的。

HashMap需要一個hash函式，它使用hashCode()和equals()方法來向集合/從集合新增和檢索元素。

當呼叫put()方法的時候，HashMap會計算key的hash值，然後把鍵值對儲存 在集合中合適的索引上。

如果key已經存在了，value會被更新成新值。

HashMap 的一些重要的特性是它的容量(capacity)，負載因子(load factor)和擴容極限 (threshold resizing)。