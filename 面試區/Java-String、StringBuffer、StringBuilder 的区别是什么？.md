---
界: Java
门: Basic
纲: 
tags: ["#Java/#Basic","#BQ"]
aliases:
  - 
date: 2021-09-06
---
- String 是字符串常量,每次操作都会生产新的对象，适用于少量字符串操作的情况
- StringBuffer 与 StringBuilder都属于字符串变量
	- StringBuffer线程安全但是性能较差
	- StringBuilder线程不安全但是性能较好


> 在单线程环境下推荐使用 StringBuilder，多线程环境下推荐使用 StringBuffer。