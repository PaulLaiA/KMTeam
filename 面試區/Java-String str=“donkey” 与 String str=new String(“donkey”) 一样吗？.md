---
界: Java
门: Basic
纲: 
tags: ["#Java/#Basic","#BQ"]
aliases:
  - 
date: 2021-09-06
---
#String 

答案 : *不一样*

# 原因
内存分配方法不一样
- String str=“donkey”
	- 分配至常量池
-  String str=new String(“donkey”)
	- 分配至堆内存中
