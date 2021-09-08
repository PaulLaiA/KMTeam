---
界: Java
门: JUC
纲: 
创建日期: 2021-09-08
---
#JUC

-   使用场景
-   synchronized底层实现原理
    -   实际上就是获得一个监视器对象monitor ，synchronized同步块使用了monitorenter和monitorexit指令实现同步，这两个指令，本质上都是对一个对象的监视器(monitor)进行获取，这个过程是排他的，也就是说同一时刻只能有一个线程获取到由synchronized所保护对象的监视器。
    -   线程执行到monitorenter指令时，会尝试获取对象所对应的monitor所有权，也就是尝试获取对象的锁，而执行monitorexit，就是释放monitor的所有权。
-   synchronized 在JDK1.6 之后做了一些优化，为了减少获得锁和释放锁来的性能开销，引入了偏向锁、轻量级锁的概念。