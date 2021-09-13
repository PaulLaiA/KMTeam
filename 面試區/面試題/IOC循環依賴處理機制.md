---
界: Spring Framework
門: Spring
綱: 
tags: ["#Spring","#BQ"]
aliases:
  - 
date: 2021-09-08
---


-   單例bean構造器參數循環依賴（無法解決）
-   原型bean循環依賴（無法解決）
-   單例bean的setter循環依賴
    -   基於java的引用傳遞，當獲得對像引用時，對象的屬性可以延後設定
    -   通過setter解決循環依賴是通過提前暴露一個ObjectFactory對像來完成，簡單來說ClassA在呼叫構造器完成對像初始化之後，在呼叫ClassA的setClassB方法之前，就會把ClassA實例化的對象通過ObjectFactory提前暴露到IOC容器中