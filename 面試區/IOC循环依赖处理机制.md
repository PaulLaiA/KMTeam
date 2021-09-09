---
界: Spring Framework
门: Spring
纲: 
tags: ["#SpringFramework/#Spring","#BQ"]
aliases:
  - 
date: 2021-09-08
---
 #Framework #SpringFramework #Spring

-   单例bean构造器参数循环依赖（无法解决）
-   原型bean循环依赖（无法解决）
-   单例bean的setter循环依赖
    -   基于java的引用传递，当获得对象引用时，对象的属性可以延后设置
    -   通过setter解决循环依赖是通过提前暴露一个ObjectFactory对象来完成，简单来说ClassA在调用构造器完成对象初始化之后，在调用ClassA的setClassB方法之前，就会把ClassA实例化的对象通过ObjectFactory提前暴露到IOC容器中