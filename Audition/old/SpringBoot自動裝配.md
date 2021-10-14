---
界: Spring Framework
門: SpringBoot
綱: 
tags: ["#Spring","#SpringBoot","#BQ"]
aliases:
  - 
date: 2021-09-08
---

-   @Configuration
-   @ComponentScan
-   @EnableAutoConfiguration（自動裝配的入口）


---


	總結：通過@Import匯入AutoConfigurationImportSelector，在該類中載入META-INF/spring.factories的配置資訊。然後篩選出以EnableAutoConfiguration為key的數據，載入到IOC容器中，實現自動裝配