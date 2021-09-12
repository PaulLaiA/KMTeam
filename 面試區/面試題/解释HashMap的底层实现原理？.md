---
界: Java
门: Container
纲: 
tags: ["#Java","#Container","#BQ"]
aliases:
  - 
date: 2021-09-12
---



Java中的HashMap是以键值对(key-value)的形式存储元素的。

HashMap需要一个hash函数，它使用hashCode()和equals()方法来向集合/从集合添加和检索元素。

当调用put()方法的时候，HashMap会计算key的hash值，然后把键值对存储 在集合中合适的索引上。

如果key已经存在了，value会被更新成新值。

HashMap 的一些重要的特性是它的容量(capacity)，负载因子(load factor)和扩容极限 (threshold resizing)。