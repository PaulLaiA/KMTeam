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

-   spring容器根据配置中的bean定义实例化bean
-   使用依赖注入填充属性
-   如果实现BeanNameAware接口，工厂通过传递bean的id来调用setBeanName
-   如果实现BeanFactoryAware接口，工厂通过传递自身的实例来调用setBeanFactory
-   如果存在与bean关联的BeanPostProcessors，调用preProcessorBeforeInitialization方法
-   如果指定init放法，那么将调用它
-   如果存在与bean关联的BeanPostProcessors，则调用postPrecessAfterInitialization方法
-   如果实现DisposableBean接口，spring容器关闭时，调用destory
-   如果指定了destory方法，那么将调用它