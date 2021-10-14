---
界: Java
門: Container
綱: 
tags: ["#Java","#Container","#BQ"]
aliases:
  - 
date: 2021-09-13
---

## 共同點
ArrayList和LinkedList都实现了List接口

## 不同點

1.	ArrayList是由Array所支持的基于一个索引的数据结构，所以它提供对元 素的随机访问，复杂度为O(1)，但LinkedList存储一系列的节点数据，每个节点 都与前一个和下一个节点相连接。所以，尽管有使用索引获取元素的方法，内部 实现是从起始点开始遍历，遍历到索引的节点然后返回元素，时间复杂度为 O(n)，比ArrayList要慢。 
2.	与ArrayList相比，在LinkedList中插入、添加和删除一个元素会更快，因 为在一个元素被插入到中间的时候，不会涉及改变数组的大小，或更新索引。 
3.	LinkedList比ArrayList消耗更多的内存，因为LinkedList中的每个节点存 储了前后节点的引用。