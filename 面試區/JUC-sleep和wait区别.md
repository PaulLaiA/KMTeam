---
界: Java
门: JUC
纲: 
tags: ["#Java/#JUC","#BQ"]
aliases:
  - 
date: 2021-09-08
---
#JUC

-   sleep是Thread中的方法，wait是Object中的方法。
-   sleep方法不会释放锁，wait会释放锁，并进入等待队列。
-   sleep不依赖同步方法（synchronized），wait需要。
-   sleep方法不需要被唤醒（休眠结束退出阻塞），wait需要使用notify或notifyAll唤醒线程。
-   sleep必须捕获异常，而 wait ， notify 和 notifyAll 不需要捕获异常