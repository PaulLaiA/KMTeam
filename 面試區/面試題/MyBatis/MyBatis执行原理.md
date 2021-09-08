---
界: Framework
门: MyBatis
纲: 
创建日期: 2021-09-08
---
#Framework #MyBatis

-   获取sqlSessionFactory对象
    -   加载配置文件封装为MappedStatement对象
-   获取sqlSession对象
    -   返回DefaultSQLSession对象，包含Executor和Configuration
    -   创建Executor
-   获取接口的代理对象
    -   拿到Mapper接口的对应的MapperProxy
    -   使用MapperProxyFactory创建代理对象
    -   代理对象包含DefaultSession（Executor）
-   执行增删改查方法
    -   调用DefaultSession的增删改查
    -   创建StatementHandler对象
    -   调用StatementHandler预编译参数以及设置参数值
    -   调用StatementHandler的增删改查方法
    -   ResultSetHandler封装结果集