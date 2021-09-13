---
界: NoSQL
門: Redis
綱: 
tags: ["#NoSQL","#Redis","#BQ"]
aliases:
  - 
date: 2021-09-07
---

## 2.8之前
1. 發送SYNC命令給master請求複製數據
2. master收到命令，在後頭進行數據持久化通過bgsave產生最新rdb快照檔案
	1. 持久化期間會繼續接收客戶端請求，會把修改數據集的請求快取在記憶體中
3. 當持久化完畢后，master會把rdb檔案數據集發送給slave
4. slave會把收到的數據進行持久化產生rdb，然後載入到記憶體中。（斷開重連會全量複製）

## 2.8之後：
-   全量複製：
    -   建立socket長連線，發送psync命令
    -   接收到psync命令，執行bgsave產生rdb快照
    -   發送rdb數據給slave
    -   繼續接收客戶端請求，修改的數據快取在記憶體中，持久化完畢后將數據集發送給slave
    -   slave把收到的數據集產生完整rdb並load到記憶體
    -   master通過socket長連線持續把寫命令發送給slave，保證主從數據一致性
-   增量複製：
    -   重新建立socket長連線
    -   發送帶offset的psync的命令
    -   如果slave的offset存在快取中，則master會將快取中的slave之後的數據同步給slave，否則全量同步
    -   master通過socket長連線持續把寫命令發送給slave，保證主從數據一致性
