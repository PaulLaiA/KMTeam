---
界: Java
門: Container
綱: 
tags: ["#Java","#Container","#BQ"]
aliases:
  - 
date: 2021-09-13
---

	LinkedList操作会更快。

## 詳解
ArrayList会根据数组的首地址和长度计算新元素的位置，然后将元素存储到计 算出的位置上。 

LinkedList包含有last指针，直接将last.next执行新元素，然后将新元素赋值 给last即可。

当然两者都需要更新长度。