---
界: DataBase
门: NoSQL
纲: Redis
创建日期: 2021-09-07
---
#DB #NoSQL #Redis

-   数据结构
    
    3.0版本：
    
    常用：string（字符串）、list（列表）、set（集合）、sortedset（有序集合）、hash
    
    不常用：bitmap（位图）、geo（地理位置类型）
    
    5.0版本：3.0基础上增加了stream类型
    
-   使用场景
    
    -   string
        
        计数器：incr article:readcount:{postId}
        
        web集群session共享：spring session + redis实现session共享
        
        分布式系统全局序列号：incrby orderId 1000
        
    -   hash
        
        电商购物车：用户id为key，商品id为field，商品数量为value
        
        博客关注取消关注：用户id为key，被关注者id为field，被关注者名称为value
        
    -   list
        
        stack：lpush+lpop
        
        queue：lpush+rpop
        
        blocking mq：lpush+brpop
        
        微博消息和微信公众号消息：lpush msg:{userId} messageId
        
        查看最新微博信息：lrange msg:{userId} 0 5
        
    -   set
        
        微信抽奖小程序：
        
        参与抽奖集合：sadd key {userId}
        
        查看抽奖用户：smembers key
        
        抽取中奖者：srandmember key [scount] or spop key [count]
        
        区别：srandmember 不删除元素 spop删除元素
        
        检查用户是否抽过奖：sismember key {userId}
        
        查看抽奖总人数：scard key
        
    -   sortedset
        
        排行榜