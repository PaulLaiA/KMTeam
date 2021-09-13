---
界: Java
門: Container
綱: 
tags: ["#Java","#Container","#BQ"]
aliases:
  - 
date: 2021-09-12
---

	所有繼承了Iterable的介面都可以通過迭代器來進行運算元據
	包含三種基礎方法
	hasNext()
	next()
	remove()

## JDK 1.4

`iterator()`是定義在Collection介面上

每個Collection的實現類都會實現`iterator()`這個方法

## JDK 1.5 + 
將`iterator()`是定義在Iterable介面上

而Collection介面繼承了Iterable介面


## Iterable 設計架構
![[Pasted image 20210912163321.png]]