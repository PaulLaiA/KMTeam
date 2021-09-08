---
界: DataBase
门: NoSQL
纲: Redis
创建日期: 2021-09-07
---
#DB #NoSQL #Redis

## 2.8之前
1. 发送SYNC命令给master请求复制数据
2. master收到命令，在后头进行数据持久化通过bgsave生成最新rdb快照文件
	1. 持久化期间会继续接收客户端请求，会把修改数据集的请求缓存在内存中
3. 当持久化完毕后，master会把rdb文件数据集发送给slave
4. slave会把收到的数据进行持久化生成rdb，然后加载到内存中。（断开重连会全量复制）

## 2.8之后：
-   全量复制：
    -   建立socket长连接，发送psync命令
    -   接收到psync命令，执行bgsave生成rdb快照
    -   发送rdb数据给slave
    -   继续接收客户端请求，修改的数据缓存在内存中，持久化完毕后将数据集发送给slave
    -   slave把收到的数据集生成完整rdb并load到内存
    -   master通过socket长连接持续把写命令发送给slave，保证主从数据一致性
-   增量复制：
    -   重新建立socket长连接
    -   发送带offset的psync的命令
    -   如果slave的offset存在缓存中，则master会将缓存中的slave之后的数据同步给slave，否则全量同步
    -   master通过socket长连接持续把写命令发送给slave，保证主从数据一致性
