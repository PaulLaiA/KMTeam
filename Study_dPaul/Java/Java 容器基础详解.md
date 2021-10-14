---
界: Java
門: Container
綱: 
tags: ["#Java","#Container","#Note"]
aliases:
  - 
date: 2021-09-13
---

# 常用分類
- Collection
	- List
		- Vector
			- Stack
		- ArrayList
		- LinkedList
	- Queue
		- LinkedList
		- Priority Queue
	- Set
		- HashSet
			- LinkHashSet
		- TreeSet
- Map
	- HashMap
	- TreeMap

# Collection 接口
	對象的集合 單列集合

## List 接口
	元素按進入先後有序保存,元素可重複

| 類型       | 底層 | 存儲順序 | 線程安全 | 重複元素         | 讀  | 寫  |
| ---------- | ---- | -------- | -------- | ---------------- | --- | --- |
| LinkedList | 鏈錶 | 有序     | 不安全   | 可以存儲重複元素 | 慢  | 快  |
| ArrayList  | 數組 | 有序     | 不安全   | 可以存儲重複類型 | 快  | 慢  |
| Vector     | 數組 | 有序     | 安全     | 可以存儲         | 慢  | 慢  |
| Stack      |      |          |          |                  |     |     |

### 場景分析
| 場景     | ArrayList | LinkedList |
| -------- | --------- | ---------- |
| 多查少改 | 推薦      |            |
| 多改少查 |           | 推薦       | 

### 小結

-	ArrayList	
	-	優點 
		1. 因為基於動態數組所以地址連續=>查詢效率高
	-	缺點
		1.	因為地址連續所以移動資料較為複雜=>插刪效率低
-	LinkedList
	-	優點
		1.	基於鏈錶的數據結構所以地址不連續=>增刪插效率高
	-	缺點
		1.	基於鏈錶的數據結構所以查詢要移動指針=>查詢效率較低

## Set 接口
	實現內部排序,元素不可重複

| 類型          | 底層         | 存儲順序 | 線程安全 | 重複元素            | 讀  | 寫  |
| ------------- | ------------ | -------- | -------- | ------------------- | --- | --- |
| HashSet       | HashMap實現  | 無序     | 不安全   | 不可重複,可以有Null | 慢  | 快  |
| LinkedHashSet | 鏈錶+HashMap | 先進先出 | 不安全   | 不可重複            | 慢  | 快  |
| TreeSet       | 二叉樹       | 元素排序 | 不安全   | 不可重複            | 慢  | 快  | 


### 場景分析
| 場景       | HashSet | TreeSet |
| ---------- | ------- | ------- |
| 需要排序   |         | 推薦    |
| 不需要排序 | 推薦    |         |

### 小結
|                | List                      | Set                              |
| -------------- | ------------------------- | -------------------------------- |
| 繼承類         | Collection                | Collection                       |
| 有序性         | 有序:插入順序決定位置     | 無序:元素位置固定,有hashCode決定 |
| 唯一性         | 可重複                    | 不可重複                         |
| 迭代元素的方法 | For循環/下標取值/迭代器   | 迭代器                           |
| 檢索速度       | 高效                      | 低效                             |
| 增刪操作       | 低效:引起後續元素位置改變 | 高效:不會引起元素位置改變                                 |

![[Java 容器基础详解.png]]

#  Map 接口

無

# 相關面試題

```dataview
table 界,門,綱,aliases
from "面試區"
where contains(file.name,"List")
or contains(file.name,"Set") 
or contains(file.name,"Map")
or contains(file.name,"Vector")
sort 界,門,綱
```