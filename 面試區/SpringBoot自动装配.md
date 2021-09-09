---
界: Spring Framework
门: SpringBoot
纲: 
tags: ["#SpringFramework/#SpringBoot","#BQ"]
aliases:
  - 
date: 2021-09-08
---
 #Framework #SpringFramework #SpringBoot

-   @Configuration
-   @ComponentScan
-   @EnableAutoConfiguration（自动装配的入口）


---


	总结：通过@Import导入AutoConfigurationImportSelector，在该类中加载META-INF/spring.factories的配置信息。然后筛选出以EnableAutoConfiguration为key的数据，加载到IOC容器中，实现自动装配