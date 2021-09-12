---
界: Java
门: Container
纲: 
tags: ["#Java","#Container","#BQ"]
aliases:
  - 
date: 2021-09-12
---

	所有继承了Iterable的介面都可以通过迭代器来进行操作数据
	包含三种基础方法
	hasNext()
	next()
	remove()

## JDK 1.4

`iterator()`是定义在Collection介面上

每个Collection的实现类都会实现`iterator()`这个方法

## JDK 1.5 + 
将`iterator()`是定义在Iterable介面上

而Collection介面继承了Iterable介面


## Iterable 设计架构
![[Pasted image 20210912163321.png]]