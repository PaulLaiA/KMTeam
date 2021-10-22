---
界: Spring Framework
门: Spring
纲: 
tags: ["#SpringFramework/#Spring","#IOC","#AOP","#Note"]
aliases:
  - 
parentNote: 
  - "[[第二部分 核心思想]]"
childrenNote: 
  - "[[第四部分 Spring IOC 應用]]"
date: {{DATE:YYYY-MM-DD}}
---

# 第六部分：Spring AOP 應用
AOP本質：在不改變原有業務邏輯的情況下增強橫切邏輯，橫切邏輯代碼往往是權限校驗代碼、⽇志代碼、事務控制代碼、性能監控代碼。

## 第1節 AOP 相關術語
### 1.1 業務主線
在講解AOP術語之前，我們先來看⼀下下⾯這兩張圖，它們就是第三部分案例需求的擴展（針對這些擴展的需求，我們只進⾏分析，在此基礎上去進⼀步回顧AOP，不進⾏實現）
![[Pasted image 20211016140856.png]]

上圖描述的就是未採⽤AOP思想設計的程序，當我們紅⾊框中圈定的⽅法時，會帶來⼤量的重複勞動。程序中充斥著⼤量的重複代碼，使我們程序的獨⽴性很差。 ⽽下圖中是採⽤了AOP思想設計的程序，它把紅框部分的代碼抽取出來的同時，運⽤動態代理技術，在運⾏期對需要使⽤的業務邏輯⽅法進⾏增強。
![[Pasted image 20211016142059.png]]

### 1.2 AOP 術語
![[Pasted image 20211016142136.png]]

連接點：⽅法開始時、結束時、正常運⾏完畢時、⽅法異常時等這些特殊的時機點，我們稱之為連接點，項⽬中每個⽅法都有連接點，連接點是⼀種候選點

切⼊點：指定AOP思想想要影響的具體⽅法是哪些，描述感興趣的⽅法

Advice增強：

第⼀個層次：指的是橫切邏輯

第⼆個層次：⽅位點（在某⼀些連接點上加⼊橫切邏輯，那麼這些連接點就叫做⽅位點，描述的是具體的特殊時機）

Aspect切⾯：切⾯概念是對上述概念的⼀個綜合

Aspect切⾯= 切⼊點+增強

= 切⼊點（鎖定⽅法） + ⽅位點（鎖定⽅法中的特殊時機）+ 橫切邏輯

**眾多的概念，⽬的就是為了鎖定要在哪個地⽅插⼊什麼橫切邏輯代碼**

## 第2節 Spring中AOP的代理選擇
Spring 實現AOP思想使⽤的是動態代理技術

默認情況下，Spring會根據被代理對像是否實現接⼝來選擇使⽤JDK還是CGLIB。當被代理對像沒有實現任何接⼝時，Spring會選擇CGLIB。當被代理對象實現了接⼝，Spring會選擇JDK官⽅的代理技術，不過我們可以通過配置的⽅式，讓Spring強制使⽤CGLIB。

## 第3節 Spring中AOP的配置⽅式

在Spring的AOP配置中，也和IoC配置⼀樣，⽀持3類配置⽅式。

第⼀類：使⽤XML配置

第⼆類：使⽤XML+註解組合配置

第三類：使⽤純註解配置

## 第4節 Spring中AOP實現
需求：橫切邏輯代碼是打印⽇志，希望把打印⽇志的邏輯織⼊到⽬標⽅法的特定位置(service層transfer⽅法)

### 4.1 XML 模式
Spring是模塊化開發的框架，使⽤aop就引⼊aop的jar
- 座標
	```xml
	<dependency>
	 <groupId>org.springframework</groupId>
	 <artifactId>spring-aop</artifactId>
	 <version>5.1.12.RELEASE</version>
	</dependency>
	<dependency>
	 <groupId>org.aspectj</groupId>
	 <artifactId>aspectjweaver</artifactId>
	 <version>1.9.4</version>
	</dependency>
	```

- AOP核心配置
	```xml
	<!--
	 Spring基于XML的AOP配置前期准备：
	 在spring的配置⽂件中加⼊aop的约束
	 xmlns:aop="http://www.springframework.org/schema/aop"
	 http://www.springframework.org/schema/aop 
	https://www.springframework.org/schema/aop/spring-aop.xsd 

	 Spring基于XML的AOP配置步骤：
	 第⼀步：把通知Bean交给Spring管理
	 第⼆步：使⽤aop:config开始aop的配置
	 第三步：使⽤aop:aspect配置切⾯
	 第四步：使⽤对应的标签配置通知的类型
	 ⼊⻔案例采⽤前置通知，标签为aop:before
	-->
	<!--把通知bean交给spring来管理-->
	<bean id="logUtil" class="com.lagou.utils.LogUtil"></bean>
	<!--开始aop的配置-->
	<aop:config>
	<!--配置切⾯-->
	 <aop:aspect id="logAdvice" ref="logUtil">
	 <!--配置前置通知-->
	 <aop:before method="printLog" pointcut="execution(public *
	com.lagou.service.impl.TransferServiceImpl.updateAccountByCardNo(com.lagou
	.pojo.Account))"></aop:before>
	 </aop:aspect>
	</aop:config>
	```

- 細節
	- 關於切入點表達式
			
				上述配置實現了對 TransferServiceImpl 的 updateAccountByCardNo ⽅法進⾏增強，在其執⾏之前，輸出了記錄⽇志的語句。這⾥⾯，我們接觸了⼀個⽐較陌⽣的名稱：切⼊點表達式，它是做什麼的呢？我們往下看。
		- 概念及作⽤
			
					切⼊點表達式，也稱之為AspectJ切⼊點表達式，指的是遵循特定語法結構的字符串，其作⽤是⽤於對符合語法格式的連接點進⾏增強。它是AspectJ表達式的⼀部分。
		- 關於AspectJ

					AspectJ是⼀個基於Java語⾔的AOP框架，Spring框架從2.0版本之後集成了AspectJ框架中切⼊點表達式的部分，開始⽀持AspectJ切⼊點表達式。
		- 切⼊點表達式使⽤示例
					

			```xml
			全限定⽅法名 访问修饰符 返回值 包名.包名.包名.类名.⽅法名(参数列表) 全匹配⽅式：
			 public void com.lagou.service.impl.TransferServiceImpl.updateAccountByCardNo(com.lagou.pojo.Account)
			 访问修饰符可以省略
			 void com.lagou.service.impl.TransferServiceImpl.updateAccountByCardNo(com.lagou.pojo.Account)
			 返回值可以使⽤*，表示任意返回值
			 *
			com.lagou.service.impl.TransferServiceImpl.updateAccountByCardNo(com.lagou.pojo.Account)
			 包名可以使⽤.表示任意包，但是有⼏级包，必须写⼏个
			 *
			....TransferServiceImpl.updateAccountByCardNo(com.lagou.pojo.Account)
			 包名可以使⽤..表示当前包及其⼦包
			 *
			..TransferServiceImpl.updateAccountByCardNo(com.lagou.pojo.Account)
			 类名和⽅法名，都可以使⽤.表示任意类，任意⽅法
			 * ...(com.lagou.pojo.Account)
			 参数列表，可以使⽤具体类型
			 基本类型直接写类型名称 ： int
			 引⽤类型必须写全限定类名：java.lang.String
			 参数列表可以使⽤*，表示任意参数类型，但是必须有参数
			 * *..*.*(*)
			 参数列表可以使⽤..，表示有⽆参数均可。有参数可以是任意类型
			 * *..*.*(..)
			 全通配⽅式：
			 * *..*.*(..)
			```

	- 改變代理⽅式的配置

		在前⾯我們已經說了，Spring在選擇創建代理對象時，會根據被代理對象的實際情況來選擇的。被代理對象實現了接⼝，則採⽤基於接⼝的動態代理。當被代理對像沒有實現任何接⼝的時候，Spring會⾃動切換到基於⼦類的動態代理⽅式。

		但是我們都知道，⽆論被代理對像是否實現接⼝，只要不是final修飾的類都可以採⽤cglib提供的⽅式創建代理對象。所以Spring也考慮到了這個情況，提供了配置的⽅式實現強制使⽤基於⼦類的動態代理（即cglib的⽅式），配置的⽅式有兩種

		- 使⽤aop:config標籤配置
			
			```xml
			<aop:config proxy-target-class="true">
			```

		- 使⽤aop:aspectj-autoproxy標籤配
			```xml
			<!--此标签是基于XML和注解组合配置AOP时的必备标签，表示Spring开启注解配置AOP
			的⽀持-->
			<aop:aspectj-autoproxy proxy-target-class="true"></aop:aspectjautoproxy>
			```

	- 五種通知類型
		- 前置通知
			
			配置⽅式：aop:before標籤
			```xml
			<!--
			 作⽤：
			 ⽤于配置前置通知。
			 出现位置：
			 它只能出现在aop:aspect标签内部
			 属性：
			 method：⽤于指定前置通知的⽅法名称
			 pointcut：⽤于指定切⼊点表达式
			 pointcut-ref：⽤于指定切⼊点表达式的引⽤
			-->
			<aop:before method="printLog" pointcut-ref="pointcut1">
			</aop:before>
			```

**執⾏時機**

前置通知永遠都會在切⼊點⽅法（業務核⼼⽅法）執⾏之前執⾏。

**細節**

前置通知可以獲取切⼊點⽅法的參數，並對其進⾏增強。

- 正常執⾏時通知

		配置⽅式
```xml
 <!--
 作⽤：
 ⽤于配置正常执⾏时通知
 出现位置：
 它只能出现在aop:aspect标签内部
 属性：
 method:⽤于指定后置通知的⽅法名称
 pointcut:⽤于指定切⼊点表达式
 pointcut-ref:⽤于指定切⼊点表达式的引⽤
 -->
 <aop:after-returning method="afterReturningPrintLog" pointcutref="pt1"></aop:after-returning>
```

- 異常通知

		配置⽅式
```xml
 <!--
 作⽤：
 ⽤于配置异常通知。
 出现位置：
 它只能出现在aop:aspect标签内部
 属性：
 method:⽤于指定异常通知的⽅法名称
 pointcut:⽤于指定切⼊点表达式
 pointcut-ref:⽤于指定切⼊点表达式的引⽤
 
 -->
 <aop:after-throwing method="afterThrowingPrintLog" pointcut-ref="pt1"
></aop:after-throwing>
```

**執⾏時機**

異常通知的執⾏時機是在切⼊點⽅法（業務核⼼⽅法）執⾏產⽣異常之後，異常通知執⾏。如果切⼊點⽅法執⾏沒有產⽣異常，則異常通知不會執⾏。

**細節**

異常通知不僅可以獲取切⼊點⽅法執⾏的參數，也可以獲取切⼊點⽅法執⾏產⽣的異常信息。

- 最終通知

		配置⽅式
```xml
<!--
 作⽤：
 ⽤于指定最终通知。
 出现位置：
 它只能出现在aop:aspect标签内部
 属性：
 method:⽤于指定最终通知的⽅法名称
 pointcut:⽤于指定切⼊点表达式
 pointcut-ref:⽤于指定切⼊点表达式的引⽤
-->
<aop:after method="afterPrintLog" pointcut-ref="pt1"></aop:after>

```

**執⾏時機**

最終通知的執⾏時機是在切⼊點⽅法（業務核⼼⽅法）執⾏完成之後，切⼊點⽅法返回之前執⾏。換句話說，⽆論切⼊點⽅法執⾏是否產⽣異常，它都會在返回之前執⾏。

**細節**

最終通知執⾏時，可以獲取到通知⽅法的參數。同時它可以做⼀些清理操作。

- 環繞通知

		配置⽅式
```xml
 <!--
 作⽤：
 ⽤于配置环绕通知。
 出现位置：
 它只能出现在aop:aspect标签的内部
 属性：
 method:⽤于指定环绕通知的⽅法名称
 pointcut:⽤于指定切⼊点表达式
 pointcut-ref:⽤于指定切⼊点表达式的引⽤
 
 -->
<aop:around method="aroundPrintLog" pointcut-ref="pt1"></aop:around>
```

```txt
**特别说明**
 
环绕通知，它是有别于前⾯四种通知类型外的特殊通知。前⾯四种通知（前置，后置，异常和最终）
它们都是指定何时增强的通知类型。⽽环绕通知，它是Spring框架为我们提供的⼀种可以通过编码的
⽅式，控制增强代码何时执⾏的通知类型。它⾥⾯借助的ProceedingJoinPoint接⼝及其实现类，
实现⼿动触发切⼊点⽅法的调⽤。
 
**ProceedingJoinPoint接⼝介绍**
 
类视图：
 
![image-20191205141201938]
(Spring%E9%AB%98%E7%BA%A7%E6%A1%86%E6%9E%B6%E8%AF%BE%E7%A8%8B%E8%AE%B2%E4%
B9%89.assets/image-20191205141201938.png)
```

### 4.2 XML+註解模式
- XML 中開啟 Spring 對註解 AOP 的⽀持
```xml
<!--开启spring对注解aop的⽀持-->
<aop:aspectj-autoproxy/>
```

- 示例
```java
/**
* 模拟记录⽇志
* @author 应癫
*/
@Component
@Aspect
public class LogUtil {
 /**
 * 我们在xml中已经使⽤了通⽤切⼊点表达式，供多个切⾯使⽤，那么在注解中如何使⽤呢？
 * 第⼀步：编写⼀个⽅法
 * 第⼆步：在⽅法使⽤@Pointcut注解
 * 第三步：给注解的value属性提供切⼊点表达式
 * 细节：
 * 1.在引⽤切⼊点表达式时，必须是⽅法名+()，例如"pointcut()"。
 * 2.在当前切⾯中使⽤，可以直接写⽅法名。在其他切⾯中使⽤必须是全限定⽅法名。
 */
 @Pointcut("execution(* com.lagou.service.impl.*.*(..))")
 public void pointcut(){}
 @Before("pointcut()")
 public void beforePrintLog(JoinPoint jp){
  Object[] args = jp.getArgs();
  System.out.println("前置通知：beforePrintLog，参数是："+ Arrays.toString(args));
 }
 @AfterReturning(value = "pointcut()",returning = "rtValue")
 public void afterReturningPrintLog(Object rtValue){
 	System.out.println("后置通知：afterReturningPrintLog，返回值是："+rtValue);
 }
 @AfterThrowing(value = "pointcut()",throwing = "e")
 public void afterThrowingPrintLog(Throwable e){
 	System.out.println("异常通知：afterThrowingPrintLog，异常是："+e);
 }
 @After("pointcut()")
 public void afterPrintLog(){
  System.out.println("最终通知：afterPrintLog");
 }
 /**
 * 环绕通知
 * @param pjp
 * @return
 */
 @Around("pointcut()")
 public Object aroundPrintLog(ProceedingJoinPoint pjp){
	 //定义返回值
	 Object rtValue = null;
	 try{
		 //前置通知
		 System.out.println("前置通知");
		 //1.获取参数
		 Object[] args = pjp.getArgs();
		 //2.执⾏切⼊点⽅法
		 rtValue = pjp.proceed(args);
		 //后置通知
		 System.out.println("后置通知");
	 }catch (Throwable t){
		 //异常通知
		 System.out.println("异常通知");
		 t.printStackTrace();
	 }finally {
		 //最终通知
		 System.out.println("最终通知");
	 }
	 return rtValue;
 }
}
```

### 4.3 註解模式
在使⽤註解驅動開發aop時，我們要明確的就是，是註解替換掉配置⽂件中的下⾯這⾏配置：
```xml
<!--开启spring对注解aop的⽀持-->
<aop:aspectj-autoproxy/>
```
在配置類中使⽤如下註解進⾏替換上述配置
```java
/**
* @author 应癫
*/
@Configuration
@ComponentScan("com.lagou")
@EnableAspectJAutoProxy //开启spring对注解AOP的⽀持
public class SpringConfiguration {
}
```

## 第5節 Spring 聲明式事務的⽀持
**編程式事務**：在業務代碼中添加事務控制代碼，這樣的事務控制機制就叫做編程式事務

**聲明式事務**：通過xml或者註解配置的⽅式達到事務控制的⽬的，叫做聲明式事務

### 5.1 事務回顧

#### 5.1.1 事務的概念

事務指邏輯上的⼀組操作，組成這組操作的各個單元，要么全部成功，要么全部不成功。從⽽確保了數據的準確與安全。

例如：A——B轉帳，對應於如下兩條sql語句:
```java
 /*转出账户减钱*/
 update account set money=money-100 where name=‘a’;
 /**转⼊账户加钱*/
 update account set money=money+100 where name=‘b’;

```
這兩條語句的執⾏，要么全部成功，要么全部不成功。

#### 5.1.2 事務的四⼤特性
**原⼦性（Atomicity）** 

原⼦性是指事務是⼀個不可分割的⼯作單位，事務中的操作要么都發⽣，要么都不發⽣。

從操作的⻆度來描述，事務中的各個操作要么都成功要么都失敗
**⼀致性（Consistency）** 

事務必須使數據庫從⼀個⼀致性狀態變換到另外⼀個⼀致性狀態。

例如轉賬前A有1000，B有1000。轉賬後A+B也得是2000。

⼀致性是從數據的⻆度來說的，（1000，1000） （900，1100），不應該出現（900，1000）

**隔離性（Isolation）** 

事務的隔離性是多個⽤戶並發訪問數據庫時，數據庫為每⼀個⽤戶開啟的事務，

每個事務不能被其他事務的操作數據所⼲擾，多個並發事務之間要相互隔離。

⽐如：事務1給員⼯漲⼯資2000，但是事務1尚未被提交，員⼯發起事務2查詢⼯資，發現⼯資漲了2000塊錢，讀到了事務1尚未提交的數據（臟讀）

**持久性（Durability）**
持久性是指⼀個事務⼀旦被提交，它對數據庫中數據的改變就是永久性的，接下來即使數據庫發⽣故障也不應該對其有任何影響。


#### 5.1.3 事務的隔離級別
不考慮隔離級別，會出現以下情況：（以下情況全是錯誤的），也即為隔離級別在解決事務並發問題

**臟讀**：

⼀個線程中的事務讀到了另外⼀個線程中未提交的數據。

**不可重複讀**：

⼀個線程中的事務讀到了另外⼀個線程中已經提交的update的數據（前後內容不⼀樣）

場景：

	員⼯A發起事務1，查詢⼯資，⼯資為1w，此時事務1尚未關閉

	財務⼈員發起了事務2，給員⼯A張了2000塊錢，並且提交了事務

	員⼯A通過事務1再次發起查詢請求，發現⼯資為1.2w，原來讀出來1w讀不到了，叫做不可重複讀

**虛讀（幻讀）**：

⼀個線程中的事務讀到了另外⼀個線程中已經提交的insert或者delete的數據（前後條數不⼀樣）

場景：

	事務1查詢所有⼯資為1w的員⼯的總數，查詢出來了10個⼈，此時事務尚未關閉

	事務2財務⼈員發起，新來員⼯，⼯資1w，向表中插⼊了2條數據，並且提交了事務

	事務1再次查詢⼯資為1w的員⼯個數，發現有12個⼈，⻅了⻤了

數據庫共定義了四種隔離級別：

Serializable（串⾏化）：可避免臟讀、不可重複讀、虛讀情況的發⽣。 （串⾏化） 最⾼

Repeatable read（可重複讀）：可避免臟讀、不可重複讀情況的發⽣。 (幻讀有可能發⽣) 第⼆

該機制下會對要update的⾏進⾏加鎖

Read committed（讀已提交）：可避免臟讀情況發⽣。不可重複讀和幻讀⼀定會發⽣。第三

Read uncommitted（讀未提交）：最低級別，以上情況均⽆法保證。 (讀未提交) 最低

注意：級別依次升⾼，效率依次降低

MySQL的默認隔離級別是：REPEATABLE READ

查詢當前使⽤的隔離級別： select @@tx_isolation;

設置MySQL事務的隔離級別： set session transaction isolation level xxx; （設置的是當前mysql連接會話的，並不是永久改變的）

#### 5.1.4 事務的傳播⾏為
事務往往在service層進⾏控制，如果出現service層⽅法A調⽤了另外⼀個service層⽅法B，A和B⽅法本身都已經被添加了事務控制，那麼A調⽤B的時候，就需要進⾏事務的⼀些協商，這就叫做事務的傳播⾏
為。

A調⽤B，我們站在B的⻆度來觀察來定義事務的傳播⾏為
![[Pasted image 20211016152332.png]]

## 5.2 Spring中事務的API
mybatis: sqlSession.commit();

hibernate: session.commit();

**PlatformTransactionManager**
```java
public interface PlatformTransactionManager {
 /**
 * 获取事务状态信息
 */
 TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
throws TransactionException;
 /**
 * 提交事务
 */
 void commit(TransactionStatus status) throws TransactionException;
 /**
 * 回滚事务
 */
 void rollback(TransactionStatus status) throws TransactionException;
}
```
**作⽤**
此接⼝是Spring的事務管理器核⼼接⼝。 Spring本身並不⽀持事務實現，只是負責提供標準，應⽤底層⽀持什麼樣的事務，需要提供具體實現類。此處也是策略模式的具體應⽤。在Spring框架中，也為我們
內置了⼀些具體策略，例如：DataSourceTransactionManager , HibernateTransactionManager 等等。 （ 和 HibernateTransactionManager 事務管理器在 spring-orm-5.1.12.RELEASE.jar 中）

Spring JdbcTemplate（數據庫操作⼯具）、Mybatis（mybatis-spring.jar）————>DataSourceTransactionManager

Hibernate框架 ——————> HibernateTransactionManager

DataSourceTransactionManager 歸根結底是橫切邏輯代碼，聲明式事務要做的就是使⽤Aop（動態代理）來將事務控制邏輯織⼊到業務代碼

## 5.3 Spring 聲明式事務配置
- 純xml模式
	- 導⼊jar
		```xml
		<dependency>
		 <groupId>org.springframework</groupId>
		 <artifactId>spring-context</artifactId>
		 <version>5.1.12.RELEASE</version>
		</dependency>
		<dependency>
		 <groupId>org.aspectj</groupId>
		 <artifactId>aspectjweaver</artifactId>
		 <version>1.9.4</version>
		</dependency>
		<dependency>
		 <groupId>org.springframework</groupId>
		 <artifactId>spring-jdbc</artifactId>
		 <version>5.1.12.RELEASE</version>
		</dependency>
		<dependency>
		 <groupId>org.springframework</groupId>
		 <artifactId>spring-tx</artifactId>
		 <version>5.1.12.RELEASE</version>
		</dependency>
		```
	- xml配置
		```xml
		<tx:advice id="txAdvice" transaction-manager="transactionManager">
		 <!--定制事務細節，傳播⾏為、隔離級別等-->
		 <tx:attributes>
		 <!--⼀般性配置-->
		 <tx:method name="*" read-only="false"
		propagation="REQUIRED" isolation="DEFAULT" timeout="-1"/>
		 <!--針對查詢的覆蓋性配置-->
		 <tx:method name="query*" read-only="true"
		propagation="SUPPORTS"/>
		 </tx:attributes>
		 </tx:advice>
		 <aop:config>
		 <!--advice-ref指向增強=橫切邏輯+⽅位-->
		 <aop:advisor advice-ref="txAdvice" pointcut="execution(*com.lagou.edu.service.impl.TransferServiceImpl.*(..))"/>
		 </aop:config>
		```

- 基於XML+註解
	- xml配置
		```xml
		<!--配置事务管理器-->
		<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		 <property name="dataSource" ref="dataSource"></property>
		</bean>
		<!--开启spring对注解事务的⽀持-->
		<tx:annotation-driven transaction-manager="transactionManager"/>
		```
	- 在接⼝、類或者⽅法上添加@Transactional註解
		```java
		@Transactional(readOnly = true,propagation = Propagation.SUPPORTS)
		```

- 基於純註解

	Spring基於註解驅動開發的事務控製配置，只需要把 xml 配置部分改為註解實現。只是需要⼀個	註解替換掉xml配置⽂件中的 <tx:annotation-driven transactionmanager="transactionManager"/> 配置。
	
	在 Spring 的配置類上添加 @EnableTransactionManagement 註解即可

	```java
	@EnableTransactionManagement//开启spring注解事务的⽀持
	public class SpringConfiguration {
	}
	```











