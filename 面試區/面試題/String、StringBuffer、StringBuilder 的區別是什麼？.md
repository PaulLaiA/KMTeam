---
界: Java
門: Basic
綱: 
tags: ["#Java","#BQ"]
aliases:
  - 
date: 2021-09-06
---
- String 是字串常量,每次操作都會生產新的對象，適用於少量字串操作的情況
- StringBuffer 與 StringBuilder都屬於字串變數
	- StringBuffer執行緒安全但是效能較差
	- StringBuilder執行緒不安全但是效能較好


> 在單執行緒環境下推薦使用 StringBuilder，多執行緒環境下推薦使用 StringBuffer。