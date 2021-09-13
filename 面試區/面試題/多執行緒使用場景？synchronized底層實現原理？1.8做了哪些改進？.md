---
界: Java
門: JUC
綱: 
tags: ["#JUC","#BQ"]
aliases:
  - 
date: 2021-09-08
---

-   使用場景
-   synchronized底層實現原理
    -   實際上就是獲得一個監視器對像monitor ，synchronized同步塊使用了monitorenter和monitorexit指令實現同步，這兩個指令，本質上都是對一個對象的監視器(monitor)進行獲取，這個過程是排他的，也就是說同一時刻只能有一個執行緒獲取到由synchronized所保護對象的監視器。
    -   執行緒執行到monitorenter指令時，會嘗試獲取對像所對應的monitor所有權，也就是嘗試獲取對象的鎖，而執行monitorexit，就是釋放monitor的所有權。
-   synchronized 在JDK1.6 之後做了一些優化，爲了減少獲得鎖和釋放鎖來的效能開銷，引入了偏向鎖、輕量級鎖的概念。