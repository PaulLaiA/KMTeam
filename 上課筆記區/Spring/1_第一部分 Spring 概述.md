---
界: Spring Framework
门: Spring
纲: 
tags: ["#SpringFramework/#Spring","#IOC","#AOP","#Note"]
aliases:
  - 
parentNote: 
  - 
childrenNote: 
  - "[[第二部分 核心思想]]"
date: {{DATE:YYYY-MM-DD}}
---

# 第一部分 Spring 概述
## 第 1 節 Spring 簡介

Spring 是分層的 full-stack（全棧） 輕量級開源框架，以 IoC 和 AOP 為內核，提供了展現層 Spring MVC 和業務層事務管理等眾多的企業級應⽤技術，還能整合開源世界眾多著名的第三⽅框架和類庫，已經成為使⽤最多的 Java EE 企業應⽤開源框架。

Spring 官⽅⽹址：[http://spring.io/](http://spring.io/)

我們經常說的 Spring 其實指的是Spring Framework（spring 框架）。

## 第 2 節 Spring 發展歷程

-   1997年 IBM 提出了EJB的思想； 1998年，SUN 制定開發標準規範EJB1.0； 1999年，EJB 1.1發 布； 2001年，EJB 2.0發布； 2003年，EJB 2.1發布； 2006年，EJB 3.0發布；
    
-   Rod Johnson（spring之⽗）
    
    -   Expert One-to-One J2EE Design and Development(2002) 闡述了J2EE使⽤EJB開發設計的優 點及解決⽅案
    -   Expert One-to-One J2EE Development without EJB(2004) 闡述了J2EE開發不使⽤EJB的解決 ⽅式（Spring雛形）
    
    2017 年 9 ⽉份發布了 Spring 的最新版本 Spring 5.0 通⽤版（GA）
    

## 第 3 節 Spring 的優勢

整個 Spring 優勢，傳達出⼀個信號，Spring 是⼀個綜合性，且有很強的思想性框架，每學習⼀ 天，就能體會到它的⼀些優勢。

-   **⽅便解耦，簡化開發**
    
    通過Spring提供的IoC容器，可以將對象間的依賴關係交由Spring進⾏控制，避免硬編碼所造成的 過度程序耦合。 ⽤戶也不必再為單例模式類、屬性⽂件解析等這些很底層的需求編寫代碼，可以更專注於上層的應⽤。
    
-   **AOP編程的⽀持**
    
    通過Spring的AOP功能，⽅便進⾏⾯向切⾯的編程，許多不容易⽤傳統OOP實現的功能可以通過 AOP輕鬆應付。
    
-   **聲明式事務的⽀持**
    
    @Transactional
    
    可以將我們從單調煩悶的事務管理代碼中解脫出來，通過聲明式⽅式靈活的進⾏事務的管理，提⾼開發效率和質量。
    
-   **⽅便程序的測試**
    
    可以⽤⾮容器依賴的編程⽅式進⾏⼏乎所有的測試⼯作，測試不再是昂貴的操作，⽽是隨⼿可做的事情。
    
-   **⽅便集成各種優秀框架**
    
    Spring可以降低各種框架的使⽤難度，提供了對各種優秀框架（Struts、Hibernate、Hessian、 Quartz等）的直接⽀持。
    
-   **降低JavaEE API的使⽤難度**
    
    Spring對JavaEE API（如JDBC、JavaMail、遠程調⽤等）進⾏了薄薄的封裝層，使這些API的使⽤ 難度⼤為降低。
    
-   **源碼是經典的 Java 學習範例** Spring的源代碼設計精妙、結構清晰、匠⼼獨⽤，處處體現著⼤師對Java設計模式靈活運⽤以及對Java技術的⾼深造詣。它的源代碼⽆意是Java技術的最佳實踐的範例。
    

## 第 4 節 Spring 的核心結構

Spring是⼀個分層⾮常清晰並且依賴關係、職責定位⾮常明確的輕量級框架，主要包括⼏個⼤模塊：數據處理模塊、Web模塊、AOP（Aspect Oriented Programming）/Aspects模塊、Core Container模塊和 Test 模塊，如下圖所示，Spring依靠這些基本模塊，實現了⼀個令⼈愉悅的融合了現有解決⽅案的零侵⼊的輕量級框架。

![[Spring 第一部分 - Spring 的核心結構.png]]

-   Spring核⼼容器（Core Container） 
	- 容器是Spring框架最核⼼的部分，它管理著Spring應⽤中 bean的創建、配置和管理。在該模塊中，包括了Spring bean⼯⼚，它為Spring提供了DI的功能。 基於bean⼯⼚，我們還會發現有多種Spring應⽤上下⽂的實現。所有的Spring模塊都構建於核⼼ 容器之上。
	    
-   ⾯向切⾯編程（AOP）/Aspects Spring
	- 對⾯向切⾯編程提供了豐富的⽀持。這個模塊是Spring應 ⽤系統中開發切⾯的基礎，與DI⼀樣，AOP可以幫助應⽤對象解耦。
	    
-   數據訪問與集成（Data Access/Integration）
	- Spring的JDBC和DAO模塊封裝了⼤量樣板代碼，這樣可以使得數據庫代碼變得簡潔，也可以更專 注於我們的業務，還可以避免數據庫資源釋放失敗⽽引起的問題。另外，Spring AOP為數據訪問 提供了事務管理服務，同時Spring還對ORM進⾏了集成，如Hibernate、MyBatis等。該模塊由 JDBC、Transactions、ORM、OXM 和 JMS 等模塊組成。
	    
-   Web 
	- 該模塊提供了SpringMVC框架給Web應⽤，還提供了多種構建和其它應⽤交互的遠程調⽤⽅ 案。 SpringMVC框架在Web層提升了應⽤的松耦合⽔平。
	    
-   Test 
	- 為了使得開發者能夠很⽅便的進⾏測試，Spring提供了測試模塊以致⼒於Spring應⽤的測 試。通過該模塊，Spring為使⽤Servlet、JNDI等編寫單元測試提供了⼀系列的mock對象實現。
	    

## 第 5 節 Spring 框架版本

![[Spring 第一部分 - Spring 框架版本.png]]

Spring Framework不同版本對 Jdk 的要求

![[Spring 第一部分 - Framework不同版本對 Jdk 的要求.png]]

JDK 11.0.5

IDE idea 2019

Maven 3.6x