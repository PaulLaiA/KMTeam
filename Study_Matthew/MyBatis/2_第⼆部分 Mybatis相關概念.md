---
界: Framework
門: MyBatis
綱: 
tags: ["#MyBatis","#BQ"]
aliases:
  - 
date: {{DATE:YYYY-MM-DD}}
---

# 第⼆部分：Mybatis相關概念

## 2.1 對象/關係數據庫映射（ORM）

ORM全稱Object/Relation Mapping：表示對象-關係映射的縮寫

ORM完成⾯向對象的編程語⾔到關係數據庫的映射。
當ORM框架完成映射後，程序員既可以利⽤⾯向 對象程序設計語⾔的簡單易⽤性，⼜可以利⽤關係數據庫的技術優勢。 
ORM把關係數據庫包裝成⾯向對 象的模型。 
ORM框架是⾯向對象設計語⾔與關係數據庫發展不同步時的中間解決⽅案。
採⽤ORM框架 後，應⽤程序不再直接訪問底層數據庫，⽽是以⾯向對象的⽅式來操作持久化對象，⽽ORM框架則將這 些⾯向對象的操作轉換成底層SQL操作。

ORM框架實現的效果：把對持久化對象的保存、修改、刪除 等操作，轉換為對數據庫的操作

## 2.2 Mybatis 簡介
MyBatis是⼀款優秀的基於ORM的半⾃動輕量級持久層框架，它⽀持定制化SQL、存儲過程以及⾼級映射。
MyBatis避免了⼏乎所有的JDBC代碼和⼿動設置參數以及獲取結果集。 
MyBatis可以使⽤簡單的 XML或註解來配置和映射原⽣類型、接⼝和Java的POJO （Plain Old Java Objects,普通⽼式Java對 象）為數據庫中的記錄。

## 2.3 Mybatis 歷史
Mybatis是⼀個半⾃動化的持久層框架，對開發⼈員開說，核⼼sql還是需要⾃⼰進⾏優化，sql和java編碼進⾏分離，功能邊界清晰，⼀個專注業務，⼀個專注數據。

分析圖示如下：

![[Pasted image 20210927112812.png | 1000]]

