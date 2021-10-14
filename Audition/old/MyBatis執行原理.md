---
界: Framework
門: MyBatis
綱: 
tags: ["#MyBatis","#BQ"]
aliases:
  - 
date: 2021-09-08
---

-   獲取sqlSessionFactory對像
    -   載入配置檔案封裝為MappedStatement對像
-   獲取sqlSession對像
    -   返回DefaultSQLSession對象，包含Executor和Configuration
    -   建立Executor
-   獲取介面的代理對像
    -   拿到Mapper介面的對應的MapperProxy
    -   使用MapperProxyFactory建立代理對像
    -   代理對像包含DefaultSession（Executor）
-   執行增刪改查方法
    -   呼叫DefaultSession的增刪改查
    -   建立StatementHandler對像
    -   呼叫StatementHandler預編譯參數以及設定參數值
    -   呼叫StatementHandler的增刪改查方法
    -   ResultSetHandler封裝結果集