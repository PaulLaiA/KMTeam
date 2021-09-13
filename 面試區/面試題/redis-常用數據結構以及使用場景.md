---
界: NoSQL
門: Redis
綱: 
tags: ["#NoSQL","#Redis","#BQ"]
aliases:
  - 
date: 2021-09-07
---

-   數據結構
    
    3.0版本：
    
    常用：string（字串）、list（列表）、set（集合）、sortedset（有序集合）、hash
    
    不常用：bitmap（點陣圖）、geo（地理位置型別）
    
    5.0版本：3.0基礎上增加了stream型別
    
-   使用場景
    
    -   string
        
        計數器：incr article:readcount:{postId}
        
        web集群session共享：spring session + redis實現session共享
        
        分佈式系統全域性序列號：incrby orderId 1000
        
    -   hash
        
        電商購物車：使用者id為key，商品id為field，商品數量為value
        
        部落格關注取消關注：使用者id為key，被關注者id為field，被關注者名稱為value
        
    -   list
        
        stack：lpush+lpop
        
        queue：lpush+rpop
        
        blocking mq：lpush+brpop
        
        微博訊息和微信公眾號訊息：lpush msg:{userId} messageId
        
        檢視最新微博資訊：lrange msg:{userId} 0 5
        
    -   set
        
        微信抽獎小程式：
        
        參與抽獎集合：sadd key {userId}
        
        檢視抽獎使用者：smembers key
        
        抽取中獎者：srandmember key [scount] or spop key [count]
        
        區別：srandmember 不刪除元素 spop刪除元素
        
        檢查使用者是否抽過獎：sismember key {userId}
        
        檢視抽獎總人數：scard key
        
    -   sortedset
        
        排行榜