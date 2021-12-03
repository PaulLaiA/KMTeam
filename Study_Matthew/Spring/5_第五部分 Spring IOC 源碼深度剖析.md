---
date: 2021-11-24
aliases: []
---

# Metadata

**Title** :: Spring

**Author** :: #Matthew 

**Classification** :: #Learn #Java #Basic #Spring

**Status** :: #🌱

**Type** :: #Note

**Previous** ::[[Study_Matthew/Spring/4_第四部分 Spring IOC 應用]]

**ParentNode** :: [[Study_Matthew/Spring/Spring|Spring]]

---



# 第五部分：Spring IOC 源碼深度剖析

- 好處：提⾼培養代碼架構思維、深⼊理解框架
	
- 原則
	
	定焦原則：抓主線
	
	宏觀原則：站在上帝視⻆，關注源碼結構和業務流程（淡化具體某⾏代碼的編寫細節）
- 讀源碼的⽅法和技巧
	
	斷點（觀察調⽤棧）
	
	反調（Find Usages）
	
	經驗（spring 框架中 doXXX，做具體處理的地⽅）
	
- Spring 源碼構建
	
	下載源碼（github）
	
	安裝 gradle 5.6.3（類似於 maven） Idea 2019.1 Jdk 11.0.5
	
	導⼊（耗費⼀定時間）
	
	編譯⼯程（順序：core-oxm-context-beans-aspects-aop）
	
	⼯程—>tasks—>compileTestJava

## 第 1 節 Spring IoC 容器初始化主體流程

### 1.1 Spring IoC 的容器體系

IoC 容器是 Spring 的核⼼模塊，是抽象了對像管理、依賴關係管理的框架解決⽅案。 

Spring 提供了很多的容器，其中 BeanFactory 是頂層容器（根容器），不能被實例化，它定義了所有 IoC 容器必須遵從的⼀套原則，具體的容器實現可以增加額外的功能，

⽐如我們常⽤到的 ApplicationContext，其下更具體的實現如 ClassPathXmlApplicationContext 包含了解析 xml 等⼀系列的內容，AnnotationConfigApplicationContext 則是包含了註解解析等⼀系列的內容。 

Spring IoC 容器繼承體系⾮常聰明，需要使⽤哪個層次⽤哪個層次即可，不必使⽤功能⼤⽽全的。

BeanFactory 頂級接⼝⽅法棧如下

![[Pasted image 20211009211933.png]]

BeanFactory 容器繼承體系

![[Pasted image 20211009212057.png]]

通過其接⼝設計，我們可以看到我們⼀貫使⽤的 ApplicationContext 除了繼承 BeanFactory 的⼦接⼝，還繼承了 ResourceLoader、MessageSource 等接⼝，因此其提供的功能也就更豐富了。

下⾯我們以 ClasspathXmlApplicationContext 為例，深⼊源碼說明 IoC 容器的初始化流程。

### 1.2 Bean ⽣命週期關鍵時機點
思路：創建⼀個類 LagouBean ，讓其實現⼏個特殊的接⼝，並分別在接⼝實現的構造器、接⼝⽅法中

斷點，觀察線程調⽤棧，分析出 Bean 對象創建和管理關鍵點的觸發時機。

LagouBean 類

```java
package com.lagou;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;
/**
* @Author 应癫
* @create 2019/12/3 11:46
*/
public class LagouBean implements InitializingBean{
 /**
 * 构造函数
 */
 public LagouBean(){
  System.out.println("LagouBean 构造器...");
 }
 /**
 * InitializingBean 接⼝实现
 */
 public void afterPropertiesSet() throws Exception {
  System.out.println("LagouBean afterPropertiesSet...");
 }
}
```

BeanPostProcessor 接⼝實現類
```java
package com.lagou;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;
/**
* @Author 应癫
* @create 2019/12/3 16:59
*/
public class MyBeanPostProcessor implements BeanPostProcessor {
 public MyBeanPostProcessor() {
  System.out.println("BeanPostProcessor 实现类构造函数...");
 }
 @Override
 public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
  if("lagouBean".equals(beanName)) {
   System.out.println("BeanPostProcessor 实现类 postProcessBeforeInitialization ⽅法被调⽤中......");
  }
  return bean;
 }
 @Override
 public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
  if("lagouBean".equals(beanName)) {
   System.out.println("BeanPostProcessor 实现类 postProcessAfterInitialization ⽅法被调⽤中......");
  }
  return bean;
 }
}
```

BeanFactoryPostProcessor 接⼝實現類

```java
package com.lagou;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;
/**
* @Author 应癫
* @create 2019/12/3 16:56
*/
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
 public MyBeanFactoryPostProcessor() {
  System.out.println("BeanFactoryPostProcessor 的实现类构造函数...");
 }
 @Override
 public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
  System.out.println("BeanFactoryPostProcessor 的实现⽅法调⽤中......");
 }
}
```

applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
 <beans xmlns="http://www.springframework.org/schema/beans"
 	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 	xsi:schemaLocation="
 		http://www.springframework.org/schema/beans
 		https://www.springframework.org/schema/beans/spring-beans.xsd">
 <bean id="lagouBean" class="com.lagou.LagouBean"/>
 <bean id="myBeanFactoryPostProcessor" class="com.lagou.MyBeanFactoryPostProcessor"/>
 <bean id="myBeanPostProcessor" class="com.lagou.MyBeanPostProcessor"/>
</beans>
```

IoC 容器源碼分析⽤例
```java
/**
* Ioc 容器源码分析基础案例
*/
@Test
public void testIoC() {
 ApplicationContext applicationContext = new  ClassPathXmlApplicationContext("classpath:applicationContext.xml");
 LagouBean lagouBean = applicationContext.getBean(LagouBean.class);
 System.out.println(lagouBean);
}
```

（1）分析 Bean 的創建是在容器初始化時還是在 getBean 時

![[Pasted image 20211009212831.png]]
根據斷點調試，我們發現，在未設置延遲加載的前提下，Bean 的創建是在容器初始化過程中完成的。

（2）分析構造函數調⽤情況

![[Pasted image 20211009212908.png]]
![[Pasted image 20211009212934.png]]

通過如上觀察，我們發現構造函數的調⽤時機在 AbstractApplicationContext 類 refresh ⽅法的 finishBeanFactoryInitialization(beanFactory)處;

（3）分析 InitializingBean 之 afterPropertiesSet 初始化⽅法調⽤情況

![[Pasted image 20211009213016.png]]
觀察調⽤棧
![[Pasted image 20211009213039.png]]
通過如上觀察，我們發現 InitializingBean 中 afterPropertiesSet ⽅法的調⽤時機也是在
AbstractApplicationContext 類 refresh ⽅法的 finishBeanFactoryInitialization(beanFactory);

（4）分析 BeanFactoryPostProcessor 初始化和調⽤情況

分別在構造函數、postProcessBeanFactory ⽅法處打斷點，觀察調⽤棧，發現 BeanFactoryPostProcessor 初始化在 AbstractApplicationContext 類 refresh ⽅法的 invokeBeanFactoryPostProcessors(beanFactory);

postProcessBeanFactory 調⽤在 AbstractApplicationContext 類 refresh ⽅法的 invokeBeanFactoryPostProcessors(beanFactory);

（5）分析 BeanPostProcessor 初始化和調⽤情況

分別在構造函數、postProcessBeanFactory ⽅法處打斷點，觀察調⽤棧，發現 BeanPostProcessor 初始化在 AbstractApplicationContext 類 refresh ⽅法的 registerBeanPostProcessors(beanFactory);

postProcessBeforeInitialization 調⽤在 AbstractApplicationContext 類 refresh ⽅法的 finishBeanFactoryInitialization(beanFactory);

postProcessAfterInitialization 調⽤在 AbstractApplicationContext 類 refresh ⽅法的 finishBeanFactoryInitialization(beanFactory);

（6）總結

根據上⾯的調試分析，我們發現 Bean 對象創建的⼏個關鍵時機點代碼層級的調⽤都在 AbstractApplicationContext 類 的 refresh ⽅法中，可⻅這個⽅法對於 Spring IoC 容器初始化來說相當
關鍵，匯總如下：
![[Pasted image 20211009213338.png]]

### 1.3 Spring IoC 容器初始化主流程

由上分析可知，Spring IoC 容器初始化的關鍵環節就在 AbstractApplicationContext#refresh() ⽅法中，我們查看 refresh ⽅法來俯瞰容器創建的主體流程，主體流程下的具體⼦流程我們後⾯再來討論。
```java
@Override
public void refresh() throws BeansException, IllegalStateException {
 synchronized (this.startupShutdownMonitor) {
	 // 第⼀步：刷新前的预处理
	 prepareRefresh();
	 /*
	 第⼆步：
	 获取 BeanFactory；默认实现是 DefaultListableBeanFactory
	 加载 BeanDefition 并注册到 BeanDefitionRegistry
	 */
	 ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
	 // 第三步：BeanFactory 的预准备⼯作（BeanFactory 进⾏⼀些设置，⽐如 context 的类加载器等）
	 prepareBeanFactory(beanFactory);
	 try {
		 // 第四步：BeanFactory 准备⼯作完成后进⾏的后置处理⼯作
		 postProcessBeanFactory(beanFactory);
		 // 第五步：实例化并调⽤实现了 BeanFactoryPostProcessor 接⼝的 Bean
		 invokeBeanFactoryPostProcessors(beanFactory);
		 // 第六步：注册 BeanPostProcessor（Bean 的后置处理器），在创建 bean 的前后等执⾏
		 registerBeanPostProcessors(beanFactory);
		 // 第七步：初始化 MessageSource 组件（做国际化功能；消息绑定，消息解析）；
		 initMessageSource();
		 // 第⼋步：初始化事件派发器
		 initApplicationEventMulticaster();
		 // 第九步：⼦类重写这个⽅法，在容器刷新的时候可以⾃定义逻辑
		 onRefresh();
		 // 第⼗步：注册应⽤的监听器。就是注册实现了 ApplicationListener 接⼝的监听器 bean
		 registerListeners();
		 /*
		 第⼗⼀步：
		 初始化所有剩下的⾮懒加载的单例 bean
		 初始化创建⾮懒加载⽅式的单例 Bean 实例（未设置属性）
		 填充属性
		 初始化⽅法调⽤（⽐如调⽤ afterPropertiesSet ⽅法、init-method ⽅法）
		 调⽤ BeanPostProcessor（后置处理器）对实例 bean 进⾏后置处
		 */
		 finishBeanFactoryInitialization(beanFactory);
		 /*
		 第⼗⼆步：
		 完成 context 的刷新。主要是调⽤ LifecycleProcessor 的 onRefresh()⽅法，并且发布事件 （ContextRefreshedEvent）
		 */
		 finishRefresh();
	 }
	 ......
 }
}

```

## 第 2 節 BeanFactory 創建流程
### 2.1 獲取 BeanFactory ⼦流程
時序圖如下
![[Pasted image 20211009213831.png]]

### 2.2 BeanDefinition 加載解析及註冊⼦流程
**（1）該⼦流程涉及到如下⼏個關鍵步驟**

**Resource 定位**：指對 BeanDefinition 的資源定位過程。通俗講就是找到定義 Javabean 信息的 XML ⽂
件，並將其封裝成 Resource 對象。

**BeanDefinition 載⼊** ：把⽤戶定義好的 Javabean 表示為 IoC 容器內部的數據結構，這個容器內部的數
據結構就是 BeanDefinition。

**註冊 BeanDefinition 到 IoC 容器**

**（2）過程分析**

**Step 1**：⼦流程⼊⼝在 AbstractRefreshableApplicationContext#refreshBeanFactory ⽅法中
![[Pasted image 20211009214036.png]]

**Step 2**：依次調⽤多個類的 loadBeanDefinitions ⽅法 —> AbstractXmlApplicationContext —>AbstractBeanDefinitionReader —> XmlBeanDefinitionReader ⼀直執⾏到 XmlBeanDefinitionReader 的 doLoadBeanDefinitions ⽅法
![[Pasted image 20211009214304.png]]

**Step 3**：我們重點觀察 XmlBeanDefinitionReader 類的 registerBeanDefinitions ⽅法，期間產⽣了多次重載調⽤，我們定位到最後⼀個
![[Pasted image 20211009214205.png]]

此處我們關注兩個地⽅：⼀個 createRederContext ⽅法，⼀個是 DefaultBeanDefinitionDocumentReader 類的 registerBeanDefinitions ⽅法，先進⼊ createRederContext ⽅法看看
![[Pasted image 20211009214410.png]]

我們可以看到，此處 Spring ⾸先完成了 NamespaceHandlerResolver 的初始化。

我們再進⼊ registerBeanDefinitions ⽅法中追踪，調⽤了 DefaultBeanDefinitionDocumentReader#registerBeanDefinitions ⽅法
![[Pasted image 20211009214445.png]]

進⼊ parseBeanDefinitions ⽅法
![[Pasted image 20211009214518.png]]

進⼊ parseDefaultElement ⽅法
![[Pasted image 20211009214550.png]]

進⼊ processBeanDefinition ⽅法
![[Pasted image 20211009214618.png]]

⾄此，註冊流程結束，我們發現，所謂的註冊就是把封裝的 XML 中定義的 Bean 信息封裝為 BeanDefinition 對象之後放⼊⼀個 Map 中，BeanFactory 是以 Map 的結構組織這些 BeanDefinition 的。
![[Pasted image 20211009214653.png]]

可以在 DefaultListableBeanFactory 中看到此 Map 的定義
```java
/** Map of bean definition objects, keyed by bean name. */
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```

**（3）時序圖**
![[Pasted image 20211009214907.png]]


## 第 3 節 Bean 創建流程
- 通過最開始的關鍵時機點分析，我們知道 Bean 創建⼦流程⼊⼝在 AbstractApplicationContext#refresh()⽅法的 finishBeanFactoryInitialization(beanFactory) 處
![[Pasted image 20211009215006.png]]

- 進⼊ finishBeanFactoryInitialization
![[Pasted image 20211009215039.png]]

- 繼續進⼊ DefaultListableBeanFactory 類的 preInstantiateSingletons ⽅法，我們找到下⾯部分的代碼，看到⼯⼚ Bean 或者普通 Bean，最終都是通過 getBean 的⽅法獲取實例
![[Pasted image 20211009215125.png]]

- 繼續跟踪下去，我們進⼊到了 AbstractBeanFactory 類的 doGetBean ⽅法，這個⽅法中的代碼很多，我們直接找到核⼼部分
![[Pasted image 20211009215211.png]]

- 接着进⼊到 AbstractAutowireCapableBeanFactory 類的⽅法，找到以下代碼部分
![[Pasted image 20211009215247.png]]

- 進⼊ doCreateBean ⽅法看看，該⽅法我們關注兩塊重點區域

	創建 Bean 實例，此時尚未設置屬性
![[Pasted image 20211009215340.png]]

## 第 4 節 lazy-init 延遲加載機制原理

- lazy-init 延遲加載機制分析

	普通 Bean 的初始化是在容器啟動初始化階段執⾏的，⽽被 lazy-init=true 修飾的 bean 則是在從容器⾥第⼀次進⾏ context.getBean() 時進⾏觸發。 Spring 啟動的時候會把所有 bean 信息(包括 XML 和註解)解
	析轉化成 Spring 能夠識別的 BeanDefinition 並存到 Hashmap ⾥供下⾯的初始化時⽤，然後對每個 BeanDefinition 進⾏處理，如果是懶加載的則在容器初始化階段不處理，其他的則在容器初始化階段進⾏初始化並依賴注⼊。

	```java
	public void preInstantiateSingletons() throws BeansException {
	 // 所有 beanDefinition 集合
	 List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);
	 // 触发所有⾮懒加载单例 bean 的初始化
	 for (String beanName : beanNames) {
		 // 获取 bean 定义
		 RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
		 // 判断是否是懒加载单例 bean，如果是单例的并且不是懒加载的则在容器创建时初始化
		 if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
			// 判断是否是 FactoryBean
			if (isFactoryBean(beanName)) {
				final FactoryBean<?> factory = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
				boolean isEagerInit;
				if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
					isEagerInit = AccessController.doPrivileged(new PrivilegedAction<Boolean>() {
					@Override
					public Boolean run() {
						return ((SmartFactoryBean<?>) factory).isEagerInit();
					}
				}, getAccessControlContext());
			}
		 }else {
			 /*
			 如果是普通 bean 则进⾏初始化并依赖注⼊，此 getBean(beanName)接下来触发的逻辑
			和
			 懒加载时 context.getBean("beanName") 所触发的逻辑是⼀样的
			 */
			 getBean(beanName);
		 }
		}
	 }
	}
	```

- 總結

	對於被修飾為 lazy-init 的 bean Spring 容器初始化階段不會進⾏ init 並且依賴注⼊，當第⼀次進⾏ getBean 時候才進⾏初始化並依賴注⼊

	對於⾮懶加載的 bean，getBean 的時候會從緩存⾥頭獲取，因為容器初始化階段 Bean 已經初始化完成並緩存了起來


## 第 5 節 Spring IoC 循環依賴問題
### 5.1 什麼是循環依賴
循環依賴其實就是循環引⽤，也就是兩個或者兩個以上的 Bean 互相持有對⽅，最終形成閉環。 ⽐如 A 依賴於 B，B 依賴於 C，C ⼜依賴於 A。
![[Pasted image 20211009215941.png]]

注意，這⾥不是函數的循環調⽤，是對象的相互依賴關係。循環調⽤其實就是⼀個死循環，除⾮有終結條件。

Spring 中循環依賴場景有：

- 構造器的循環依賴（構造器注⼊）
- Field 屬性的循環依賴（set 注⼊）

其中，構造器的循環依賴問題⽆法解決，只能拋出 BeanCurrentlyInCreationException 異常，在解決屬性循環依賴時，spring 採⽤的是提前暴露對象的⽅法。

### 5.2 循環依賴處理機制
- 單例 bean 構造器參數循環依賴（⽆法解決）
- prototype 原型 bean 循環依賴（⽆法解決）
	
	對於原型 bean 的初始化過程中不論是通過構造器參數循環依賴還是通過 setXxx ⽅法產⽣循環依賴，Spring 都 會直接報錯處理。

	AbstractBeanFactory.doGetBean()⽅法：
	```java
	if (isPrototypeCurrentlyInCreation(beanName)) {
	 throw new BeanCurrentlyInCreationException(beanName);
	}
	```

	```java
	protected boolean isPrototypeCurrentlyInCreation(String beanName) {
	 Object curVal = this.prototypesCurrentlyInCreation.get();
	 return (curVal != null && (curVal.equals(beanName) || (curVal instanceof Set && ((Set<?>) curVal).contains(beanName))));
	}
	```

	在獲取 bean 之前如果這個原型 bean 正在被創建則直接拋出異常。原型 bean 在創建之前會進⾏標記這個 beanName 正在被創建，等創建結束之後會刪除標記
	```java
	try {
	 //创建原型 bean 之前添加标记
	 beforePrototypeCreation(beanName);
	 //创建原型 bean
	 prototypeInstance = createBean(beanName, mbd, args);
	}
	finally {
	 //创建原型 bean 之后删除标记
	 afterPrototypeCreation(beanName);
	}
	```

**總結：Spring 不⽀持原型 bean 的循環依賴。**

- 單例 bean 通過 setXxx 或者@Autowired 進⾏循環依賴

	Spring 的循環依賴的理論依據基於 Java 的引⽤傳遞，當獲得對象的引⽤時，對象的屬性是可以延後設置的，但是構造器必須是在獲取引⽤之前 Spring 通過 setXxx 或者@Autowired ⽅法解決循環依賴其實是通過提前暴露⼀個 ObjectFactory 對象來完成的，簡單來說 ClassA 在調⽤構造器完成對像初始化之後，在調⽤ ClassA 的 setClassB ⽅法之前就把 ClassA 實例化的對象通過 ObjectFactory 提前暴露到 Spring 容器中。

	Spring 容器初始化 ClassA 通過構造器初始化對像後提前暴露到 Spring 容器。
	```java
	boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences && isSingletonCurrentlyInCreation(beanName));
 if (earlySingletonExposure) {
	 if (logger.isDebugEnabled()) {
	 	logger.debug("Eagerly caching bean '" + beanName +  "' to allow for resolving potential circular references");
	 }
	 //将初始化后的对象提前已 ObjectFactory 对象注⼊到容器中
	 addSingletonFactory(beanName, new ObjectFactory<Object>() {
	 @Override
	 public Object getObject() throws BeansException {
		 return getEarlyBeanReference(beanName, mbd, bean);
	 }
	 });
 }
	```

	- ClassA 調⽤ setClassB ⽅法，Spring ⾸先嘗試從容器中獲取 ClassB，此時 ClassB 不存在 Spring 容器中。
	- Spring 容器初始化 ClassB，同時也會將 ClassB 提前暴露到 Spring 容器中
	- ClassB 調⽤ setClassA ⽅法，Spring 從容器中獲取 ClassA ，因為第⼀步中已經提前暴露了 ClassA，因此可以獲取到 ClassA 實例
		- ClassA 通過 spring 容器獲取到 ClassB，完成了對像初始化操作。
	- 這樣 ClassA 和 ClassB 都完成了對像初始化操作，解決了循環依賴問題。





