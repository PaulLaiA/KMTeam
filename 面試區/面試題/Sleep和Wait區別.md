---
界: Java
門: JUC
綱: 
tags: ["#JUC","#BQ"]
aliases:
  - 
date: 2021-09-08
---

-   sleep是Thread中的方法，wait是Object中的方法。
-   sleep方法不會釋放鎖，wait會釋放鎖，並進入等待佇列。
-   sleep不依賴同步方法（synchronized），wait需要。
-   sleep方法不需要被喚醒（休眠結束退出阻塞），wait需要使用notify或notifyAll喚醒執行緒。
-   sleep必須捕獲異常，而 wait ， notify 和 notifyAll 不需要捕獲異常