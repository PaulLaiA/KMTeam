---
date: 2021-11-04
aliases: ["Mybatis架構原理"]
---

# Metadata

**Title** :: 自定義持久層框架

**Author** :: #Matthew 

**Classification** :: #Learn #Java #Basic

**Status** :: #🌱

**Type** :: #Note

**Previous** ::[[Study_Matthew/MyBatis/8_第八部分 Mybatis插件]]

**ParentNode** :: [[Study_Matthew/MyBatis/MyBatis|MyBatis]]

---



# 第九部分： Mybatis 架構原理

## 9.1 架構設計
![[Pasted image 20211003145609.png]]

我們把 Mybatis 的功能架構分為三層：

(1) API 接⼝層：提供給外部使⽤的接⼝ API，開發⼈員通過這些本地 API 來操縱數據庫。接⼝層⼀接收到 調⽤請求就會調⽤數據處理層來完成具體的數據處理。

	MyBatis 和數據庫的交互有兩種⽅式：
		a. 使⽤傳統的 MyBati s 提供的 API ；
		b. 使⽤ Mapper 代理的⽅式

(2) 數據處理層：負責具體的 SQL 查找、SQL 解析、SQL 執⾏和執⾏結果映射處理等。它主要的⽬的是根 據調⽤的請求完成⼀次數據庫操作。

(3) 基礎⽀撐層：負責最基礎的功能⽀撐，包括連接管理、事務管理、配置加載和緩存處理，這些都是共 ⽤的東⻄，將他們抽取出來作為最基礎的組件。為上層的數據處理層提供最基礎的⽀撐


## 9.2 主要構件及其相互關係
![[Pasted image 20211003145800.png]]

![[Pasted image 20211003145944.png]]

## 9.3 總體流程
**(1) 加載配置並初始化**

	觸發條件：加載配置⽂件
	配置來源於兩個地⽅，⼀個是配置⽂件(主配置⽂件 conf.xml,mapper ⽂件*.xml),—個是 java 代碼中的 註解，將主配置⽂件內容解析封裝到 Configuration,將 sql 的配置信息加載成為⼀個 mappedstatement 對象，存儲在內存之中
	
**(2) 接收調⽤請求**

	觸發條件：調⽤ Mybatis 提供的 API
	傳⼊參數：為 SQL 的 ID 和傳⼊參數對象
	處理過程：將請求傳遞給下層的請求處理層進⾏處理。

**(3) 處理操作請求**

	觸發條件：API 接⼝層傳遞請求過來
	傳⼊參數：為 SQL 的 ID 和傳⼊參數對象
	處理過程：
		(A) 根據 SQL 的 ID 查找對應的 MappedStatement 對象。
		(B) 根據傳⼊參數對象解析 MappedStatement 對象，得到最終要執⾏的 SQL 和執⾏傳⼊參數。
		(C) 獲取數據庫連接，根據得到的最終 SQL 語句和執⾏傳⼊參數到數據庫執⾏，並得到執⾏結果。
		(D) 根據 MappedStatement 對像中的結果映射配置對得到的執⾏結果進⾏轉換處理，並得到最終的處理 結果。
		(E) 釋放連接資源。

**(4) 返回處理結果** 

	將最終的處理結果返回。