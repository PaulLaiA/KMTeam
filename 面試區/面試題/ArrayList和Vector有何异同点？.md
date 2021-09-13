---
界: Java
門: Container
綱: 
tags: ["#Java","#Container","#BQ"]
aliases:
  - 
date: 2021-09-13
---

## 相同點
1. 两者都是基于索引的，内部由一个数组支持。 
2. 两者维护插入的顺序，我们可以根据插入顺序来获取元素。
3. ArrayList和Vector的迭代器实现都是fail-fast的。
4. ArrayList和Vector两者允许null值，也可以使用索引值对元素进行随机访 问。

## 不同點


1.	Vector是同步的，而ArrayList不是。然而，如果你寻求在迭代的时候对列 表进行改变，你应该使用CopyOnWriteArrayList。 
2.	ArrayList比Vector快，它因为有同步，不会过载。 
3.	ArrayList更加通用，因为我们可以使用Collections工具类轻易地获取同 步列表和只读列表。