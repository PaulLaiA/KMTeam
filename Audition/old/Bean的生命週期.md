---
界: Spring Framework
門: Spring
綱: 
tags: ["#Spring","#BQ"]
aliases:
  - 
date: 2021-09-08
---

-   spring容器根據配置中的bean定義實例化bean
-   使用依賴注入填充屬性
-   如果實現BeanNameAware介面，工廠通過傳遞bean的id來呼叫setBeanName
-   如果實現BeanFactoryAware介面，工廠通過傳遞自身的實例來呼叫setBeanFactory
-   如果存在與bean關聯的BeanPostProcessors，呼叫preProcessorBeforeInitialization方法
-   如果指定init放法，那麼將呼叫它
-   如果存在與bean關聯的BeanPostProcessors，則呼叫postPrecessAfterInitialization方法
-   如果實現DisposableBean介面，spring容器關閉時，呼叫destory
-   如果指定了destory方法，那麼將呼叫它