---
界: Spring Framework
门: Spring
纲: 
tags: ["#SpringFramework/#Spring","#IOC","#Note"]
aliases:
  - 
date: {{DATE:YYYY-MM-DD}}
---

# 第四部分 Spring IoC 應用

## 第 1 節 Spring IoC 基礎
![[Pasted image 20210909150107.png]]

### 1.1 BeanFactory 與 ApplicationContext 區別
BeanFactory 是 Spring 框架中 IoC 容器的頂層接⼝,它只是⽤來定義⼀些基礎功能,定義⼀些基礎規範,⽽ ApplicationContext 是它的⼀個⼦接⼝，所以 ApplicationContext 是具備 BeanFactory 提供的全部功能的。
通常，我們稱 BeanFactory 為 SpringIOC 的基礎容器，ApplicationContext 是容器的⾼級接⼝，⽐ BeanFactory 要擁有更多的功能，⽐如說國際化⽀持和資源訪問（xml，java 配置類）等等
![[Pasted image 20210909150243.png]]

啟動 Ioc 容器的方式
-  java 環境下啟動 Ioc 容器
	- ClassPathXmlApplicationContext：從類的根路徑下加載配置⽂件（推薦使⽤）
	- FileSystemXmlApplicationContext：從磁盤路徑上加載配置⽂件
	- AnnotationConfigApplicationContext：純註解模式下啟動 Spring 容器
- Web 環境下啟動 Ioc 容器
	- 從 xml 啟動容器
	```xml
	<!DOCTYPE web-app PUBLIC
	"-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
	"http://java.sun.com/dtd/web-app_2_3.dtd" >
	<web-app>
	 <display-name>Archetype Created Web Application</display-name>
	 <!--配置 Spring ioc 容器的配置⽂件-->
	 <context-param>
	 <param-name>contextConfigLocation</param-name>
	 <param-value>classpath:applicationContext.xml</param-value>
	 </context-param>
	 <!--使⽤监听器启动 Spring 的 IOC 容器-->
	 <listener>
	 <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	 </listener>
	</web-app>
	```
	- 從配置類啟動容器
```xml
	<!DOCTYPE web-app PUBLIC
	"-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
	"http://java.sun.com/dtd/web-app_2_3.dtd" >
	<web-app>
	 <display-name>Archetype Created Web Application</display-name>
	 <!--告诉 ContextloaderListener 知道我们使⽤注解的⽅式启动 ioc 容器-->
	 <context-param>
	 <param-name>contextClass</param-name>
	 <param-value>org.springframework.web.context.support.AnnotationConfigWebAppli
	cationContext</param-value>
	 </context-param>

	 <!--配置启动类的全限定类名-->
	 <context-param>
	 <param-name>contextConfigLocation</param-name>
	 <param-value>com.lagou.edu.SpringConfig</param-value>
	 </context-param>
	 <!--使⽤监听器启动 Spring 的 IOC 容器-->
	 <listener>
	 <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	 </listener>
	</web-app>
```

### 1.2 純 xml 模式
本部分內容我們不採⽤⼀⼀講解知識點的⽅式，⽽是採⽤ Spring IoC 純 xml 模式改造我們前⾯⼿寫的
IoC 和 AOP 實現，在改造的過程中，把各個知識點串起來。

- xml 文件頭
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
 https://www.springframework.org/schema/beans/spring-beans.xsd">
```

**實例化 Bean 的三種方式**
- 方式一：使用參數構造函數
	在默認情況下，它會通過反射調⽤⽆參構造函數來創建對象。如果類中沒有⽆參構造函數，將創建失敗。
```xml
	<!--配置 service 对象-->
	<bean id="userService" class="com.lagou.service.impl.TransferServiceImpl">
	</bean>
```


- 方式二：使用靜態方法創建
		
在實際開發中，我們使⽤的對像有些時候並不是直接通過構造函數就可以創建出來的，它可能在創
建的過程 中會做很多額外的操作。此時會提供⼀個創建對象的⽅法，恰好這個⽅法是 static 修飾的
⽅法，即是此種情 況。
例如，我們在做 Jdbc 操作時，會⽤到 java.sql.Connection 接⼝的實現類，如果是 MySQL 數據庫，那
麼⽤的就 是 JDBC4Connection，但是我們不會去寫 JDBC4Connection connection = new
JDBC4Connection() ，因 為我們要註冊驅動，還要提供 URL 和憑證信息，
⽤ DriverManager.getConnection ⽅法來獲取連接。
那麼在實際開發中，尤其早期的項⽬沒有使⽤ Spring 框架來管理對象的創建，但是在設計時使⽤了
⼯⼚模式 解耦，那麼當接⼊ spring 之後，⼯⼚類創建對象就具有和上述例⼦相同特徵，即可採⽤
此種⽅式配置。


   ```xml
	<!--使⽤静态⽅法创建对象的配置⽅式-->
	<bean id="userService" class="com.lagou.factory.BeanFactory"
 	factory-method="getTransferService"></bean>
   ```
   - 方式三：使用實例化方法創建
此種⽅式和上⾯靜態⽅法創建其實類似，區別是⽤於獲取對象的⽅法不再是 static 修飾的了，⽽是
類中的⼀ 個普通⽅法。此種⽅式⽐靜態⽅法創建的使⽤⼏率要⾼⼀些。
在早期開發的項⽬中，⼯⼚類中的⽅法有可能是靜態的，也有可能是⾮靜態⽅法，當是⾮靜態⽅法
時，即可 採⽤下⾯的配置⽅式：
   ```xml
	<!--使⽤实例⽅法创建对象的配置⽅式-->
	<bean id="beanFactory"
	class="com.lagou.factory.instancemethod.BeanFactory"></bean>
	<bean id="transferService" factory-bean="beanFactory" factorymethod="getTransferService"></bean>
   ```
   - Bean 的 X 及⽣命週期
	   - 作⽤範圍的改變
			在 spring 框架管理 Bean 對象的創建時，Bean 對象默認都是單例的，但是它⽀持配置的⽅式改
			變作⽤範圍。作⽤範圍官⽅提供的說明如下圖：
   			![[Pasted image 20210909152826.png]]
	在上圖中提供的這些選項中，我們實際開發中⽤到最多的作⽤範圍就是 singleton（單例模式）和
prototype（原型模式，也叫多例模式）。配置⽅式參考下⾯的代碼：
   ```xml
	<!--配置 service 对象-->
	<bean id="transferService"
	class="com.lagou.service.impl.TransferServiceImpl" scope="singleton">
	</bean>
   ```
 - 不同作用範圍的生命週期
	- **單例模式：singleton**
		- 對像出⽣：當創建容器時，對象就被創建了。
		- 對象活著：只要容器在，對象⼀直活著。
		- 對象死亡：當銷毀容器時，對象就被銷毀了。
		- ⼀句話總結：單例模式的 bean 對象⽣命週期與容器相同。
	- **多例模式：prototype**
		- 對像出⽣：當使⽤對象時，創建新的對象實例。
		- 對象活著：只要對像在使⽤中，就⼀直活著。
		- 對象死亡：當對象⻓時間不⽤時，被 java 的垃圾回收器回收了。
		- ⼀句話總結：多例模式的 bean 對象，spring 框架只負責創建，不負責銷毀。

- Bean 標籤屬性
	- 在基於 xml 的 IoC 配置中，bean 標籤是最基礎的標籤。它表示了 IoC 容器中的⼀個對象。換句話說，如果⼀個對像想讓 spring 管理，在 XML 的配置中都需要使⽤此標籤配置，Bean 標籤的屬性如下：
		- id 屬性： ⽤於給 bean 提供⼀個唯⼀標識。在⼀個標籤內部，標識必須唯⼀。
		- class 屬性：⽤於指定創建 Bean 對象的全限定類名。
		- name 屬性：⽤於給 bean 提供⼀個或多個名稱。多個名稱⽤空格分隔。
		- factory-bean 屬性：⽤於指定創建當前 bean 對象的⼯⼚ bean 的唯⼀標識。當指定了此屬性之後，class 屬性失效。
		- factory-method 屬性：⽤於指定創建當前 bean 對象的⼯⼚⽅法，如配合 factory-bean 屬性使⽤，則 class 屬性失效。如配合 class 屬性使⽤，則⽅法必須是 static 的。
		- scope 屬性：⽤於指定 bean 對象的作⽤範圍。通常情況下就是 singleton。當要⽤到多例模式時，可以配置為 prototype。
		- init-method 屬性：⽤於指定 bean 對象的初始化⽅法，此⽅法會在 bean 對象裝配後調⽤。必須是⼀個⽆參⽅法。
		- destory-method 屬性：⽤於指定 bean 對象的銷毀⽅法，此⽅法會在 bean 對象銷毀前執⾏。它只能為 scope 是 singleton 時起作⽤。

- DI 依賴注入的 xml 配置
	- 依賴注入分類
		- 按照注⼊的⽅式分類
			- 構造函數注⼊：顧名思義，就是利⽤帶參構造函數實現對類成員的數據賦值。
			- set ⽅法注⼊：它是通過類成員的 set ⽅法實現數據的注⼊。 （使⽤最多的）
		- 按照注⼊的數據類型分類
			- 基本類型和 String
				- 注⼊的數據類型是基本類型或者是字符串類型的數據。
			- 其他 Bean 類型
				- 注⼊的數據類型是對像類型，稱為其他 Bean 的原因是，這個對像是要求出現在 IoC 容器中的。那麼針對當前 Bean 來說，就是其他 Bean 了。
			- 複雜類型（集合類型）
				- 注⼊的數據類型是 Aarry，List，Set，Map，Properties 中的⼀種類型。
	- 依賴注⼊的配置實現之構造函數注⼊ 顧名思義，就是利⽤構造函數實現對類成員的賦值。它的使⽤要求是，類中提供的構造函數參數個數必須和配置的參數個數⼀致，且數據類型匹配。同時需要注意的是，當沒有⽆參構造時，則必須提供構造函數參數的注⼊，否則 Spring 框架會報錯。

![[Pasted image 20210909155410.png]]

在使⽤構造函數注⼊時，涉及的標籤是 constructor-arg ，該標籤有如下屬性：

**name**：⽤於給構造函數中指定名稱的參數賦值。

**index**：⽤於給構造函數中指定索引位置的參數賦值。

**value**：⽤於指定基本類型或者 String 類型的數據。

**ref**：⽤於指定其他 Bean 類型的數據。寫的是其他 bean 的唯⼀標識。
- 依賴注⼊的配置實現之 set ⽅法注⼊
顧名思義，就是利⽤字段的 set ⽅法實現賦值的注⼊⽅式。此種⽅式在實際開發中是使⽤最多的注
⼊⽅式。

![[Pasted image 20210909160800.png]]

在使⽤ set ⽅法注⼊時，需要使⽤ property 標籤，該標籤屬性如下：

**name**：指定注⼊時調⽤的 set ⽅法名稱。 （注：不包含 set 這三個字⺟,druid 連接池指定屬性名稱）

**value**：指定注⼊的數據。它⽀持基本類型和 String 類型。

**ref**：指定注⼊的數據。它⽀持其他 bean 類型。寫的是其他 bean 的唯⼀標識。

- 複雜數據類型注⼊ ⾸先，解釋⼀下複雜類型數據，它指的是集合類型數據。集合分為兩類，⼀類是 List 結構（數組結構），⼀類是 Map 接⼝（鍵值對） 。接下來就是注⼊的⽅式的選擇，只能在構造函數和 set ⽅法中選擇，我們的示例選⽤ set ⽅法注⼊。

![[Pasted image 20210909155559.png]]

![[Pasted image 20210909155629.png]]

在 List 結構的集合數據注⼊時， ==array== , ==list== , ==set== 這三個標籤通⽤，另外注值的 value 標籤內部
可以直接寫值，也可以使⽤ ==bean== 標籤配置⼀個對象，或者⽤ ==ref== 標籤引⽤⼀個已經配合的 bean
的唯⼀標識。

在 Map 結構的集合數據注⼊時， ==map== 標籤使⽤ ==entry== ⼦標籤實現數據注⼊，== entry== 標籤可以使
⽤**key**和**value**屬性指定存⼊ map 中的數據。使⽤**value-ref**屬性指定已經配置好的 bean 的引⽤。
同時 ==entry== 標籤中也可以使⽤ ==ref== 標籤，但是不能使⽤ ==bean== 標籤。 ⽽ ==property== 標籤中不能使
⽤ ==ref== 或者 ==bean== 標籤引⽤對象

### 1.3 xml 與註解相結合模式

注意：

1）實際企業開發中，純 xml 模式使⽤已經很少了

2）引⼊註解功能，不需要引⼊額外的 jar

3）xml+ 註解結合模式，xml ⽂件依然存在，所以，spring IOC 容器的啟動仍然從加載 xml 開始

4）哪些 bean 的定義寫在 xml 中，哪些 bean 的定義使⽤註解

**第三⽅ jar 中的 bean 定義在 xml，⽐如德魯伊數據庫連接池**

**⾃⼰開發的 bean 定義使⽤註解**

- xml 形式 對應的註解形式
![[Pasted image 20210909161517.png]]

- DI 依賴注入的註解方式

	@Autowired (推薦使用)
	
	@Autowired 為 Spring 提供的註解，需要導入包 org.springframework.beans.factory.annotation.Autowired。
	
	@Autowired 採取的策略為按照類型注入
	
```java
public class TransferServiceImpl { 
	@Autowired 
	private AccountDao accountDao; 
}
```
如上代碼所示，這樣裝配回去 spring 容器中找到類型為 AccountDao 的類，然後將其註⼊進來。這樣會產⽣⼀個問題，當⼀個類型有多個 bean 值的時候，會造成⽆法選擇具體注⼊哪⼀個的情況，這個時候我們需要配合著@Qualifier 使⽤。

@Qualifier 告訴 Spring 具體去裝配哪個對象。
```java
public class TransferServiceImpl { 
	@Autowired 
	@Qualifier(name="jdbcAccountDaoImpl") 
	private AccountDao accountDao; }
```

這個時候我們就可以通過類型和名稱定位到我們想注⼊的對象。

@Resource

@Resource 註解由 J2EE 提供，需要導⼊包 javax.annotation.Resource。

@Resource 默認按照 ByName ⾃動注⼊。
```java
public class TransferService {
 @Resource 
 private AccountDao accountDao;
 @Resource(name="studentDao") 
 private StudentDao studentDao;
 @Resource(type="TeacherDao") 
 private TeacherDao teacherDao;
 @Resource(name="manDao",type="ManDao") 
 private ManDao manDao;
}
```

- 如果同時指定了 name 和 type，則從 Spring 上下⽂中找到唯⼀匹配的 bean 進⾏裝配，找不到則拋出異常。
- 如果指定了 name，則從上下⽂中查找名稱（id）匹配的 bean 進⾏裝配，找不到則拋出異常。
- 如果指定了 type，則從上下⽂中找到類似匹配的唯⼀ bean 進⾏裝配，找不到或是找到多個，都會拋出異常。
- 如果既沒有指定 name，⼜沒有指定 type，則⾃動按照 byName ⽅式進⾏裝配；

**注意:** @Resource 在 Jdk 11 中已經移除，如果要使⽤，需要單獨引入 jar 包
```xml
<dependency>
 <groupId>javax.annotation</groupId>
 <artifactId>javax.annotation-api</artifactId>
 <version>1.3.2</version>
</dependency>
```

### 1.4 純註解模式
改造 xml+ 註解模式，將 xml 中遺留的內容全部以註解的形式遷移出去，最終刪除 xml，從 Java 配置類啟動
對應註解

@Configuration 註解，表名當前類是⼀個配置類

@ComponentScan 註解，替代 context:component-scan

@PropertySource，引⼊外部属性配置⽂件 

@Import 引⼊其他配置类 

@Value 对变量赋值，可以直接赋值，也可以使⽤ ${} 读取资源配置⽂件中的信息 

@Bean 将⽅法返回对象加⼊ SpringIOC 容器


## 第 2 節 Spring IOC 高級特性
### 2.1 lazy-Init 延遲加載
Bean 的延遲加載（延遲創建）

ApplicationContext 容器的默認⾏為是在啟動服務器時將所有 singleton bean 提前進⾏實例化。提前實例化意味著作為初始化過程的⼀部分，ApplicationContext 實例會創建並配置所有的 singletonbean。

⽐如：
```xml
<bean id="testBean" class="cn.lagou.LazyBean" />
該 bean 默認的設置為:
<bean id="testBean" calss="cn.lagou.LazyBean" lazy-init="false" />
```

lazy-init="false"，⽴即加載，表示在 spring 啟動時，⽴刻進⾏實例化。

如果不想讓⼀個 singleton bean 在 ApplicationContext 實現初始化時被提前實例化，那麼可以將 bean
設置為延遲實例化。

```xml
<bean id="testBean" calss="cn.lagou.LazyBean" lazy-init="true" />
```

設置 lazy-init 為 true 的 bean 將不會在 ApplicationContext 啟動時提前被實例化，⽽是第⼀次向容器通過 getBean 索取 bean 時實例化的。

如果⼀個設置了⽴即加載的 bean1，引⽤了⼀個延遲加載的 bean2 ，那麼 bean1 在容器啟動時被實例化，⽽ bean2 由於被 bean1 引⽤，所以也被實例化，這種情況也符合延時加載的 bean 在第⼀次調⽤時才被實例化的規則。

也可以在容器層次中通過在 元素上使⽤ "default-lazy-init" 屬性來控制延時初始化。如下⾯配置：

```xml
<beans default-lazy-init="true">
 <!-- no beans will be eagerly pre-instantiated... -->
</beans>
```

如果⼀個 bean 的 scope 屬性為 scope="pototype" 時，即使設置了 lazy-init="false"，容器啟動時也不
會實例化 bean，⽽是調⽤ getBean ⽅法實例化的。
應⽤場景
（1）開啟延遲加載⼀定程度提⾼容器啟動和運轉性能
（2）對於不常使⽤的 Bean 設置延遲加載，這樣偶爾使⽤的時候再加載，不必要從⼀開始該 Bean 就佔⽤資源


### 2.1 FactoryBean 和 BeanFactory

BeanFactory 接⼝是容器的頂級接⼝，定義了容器的⼀些基礎⾏為，負責⽣產和管理 Bean 的⼀個⼯⼚，具體使⽤它下⾯的⼦接⼝類型，⽐如 ApplicationContext；此處我們重點分析 FactoryBean

Spring 中 Bean 有兩種，⼀種是普通 Bean，⼀種是⼯⼚ Bean（FactoryBean），FactoryBean 可以⽣成某⼀個類型的 Bean 實例（返回給我們），也就是說我們可以藉助於它⾃定義 Bean 的創建過程。

Bean 創建的三種⽅式中的靜態⽅法和實例化⽅法和 FactoryBean 作⽤類似，FactoryBean 使⽤較多，尤其在 Spring 框架⼀些組件中會使⽤，還有其他框架和 Spring 框架整合時使⽤

```java
// 可以让我们⾃定义 Bean 的创建过程（完成复杂 Bean 的定义）
public interface FactoryBean<T> {
 @Nullable
 // 返回 FactoryBean 创建的 Bean 实例，如果 isSingleton 返回 true，则该实例会放到 Spring 容器
的单例对象缓存池中 Map
 T getObject() throws Exception;
 @Nullable
 // 返回 FactoryBean 创建的 Bean 类型
 Class<?> getObjectType();
 // 返回作⽤域是否单例
 default boolean isSingleton() {
 return true;
 }
}
```

Company 類
```java
package com.lagou.edu.pojo;
/**
* @author 应癫
*/
public class Company {
 private String name;
 private String address;
 private int scale;
 public String getName() {
 	return name;
 }
 public void setName(String name) {
 	this.name = name;
 }
 public String getAddress() {
 	return address;
 }
 public void setAddress(String address) {
 	this.address = address;
 }
 public int getScale() {
 	return scale;
 }
 public void setScale(int scale) {
 	this.scale = scale;
 }
 @Override
 public String toString() {
	 return "Company{" +
	 "name='" + name + '\'' +
	 ", address='" + address + '\'' +
	 ", scale=" + scale +
	 '}';
 }
}
```

CompanyFactoryBean 類
```java
package com.lagou.edu.factory;
import com.lagou.edu.pojo.Company;
import org.springframework.beans.factory.FactoryBean;
/**
* @author 应癫
*/
public class CompanyFactoryBean implements FactoryBean<Company> {
 private String companyInfo; // 公司名称,地址,规模
 public void setCompanyInfo(String companyInfo) {
 this.companyInfo = companyInfo;
 }
 @Override
 public Company getObject() throws Exception {
 // 模拟创建复杂对象 Company
 Company company = new Company();
 String[] strings = companyInfo.split(",");
 company.setName(strings[0]);
 company.setAddress(strings[1]);
 company.setScale(Integer.parseInt(strings[2]));
 return company;
 }
 @Override
 public Class<?> getObjectType() {
 return Company.class;
 }
 @Override
 public boolean isSingleton() {
 return true;
 }
}
```

xml 配置
```xml
<bean id="companyBean" class="com.lagou.edu.factory.CompanyFactoryBean">
 <property name="companyInfo" value="拉勾,中关村,500"/>
</bean>
```
測試，獲取 FactoryBean 產生的對象
```java
Object companyBean = applicationContext.getBean("companyBean");
System.out.println("bean:" + companyBean);
// 结果如下
bean:Company{name='拉勾', address='中关村', scale=500}
```

### 2.3 後置處理器

Spring 提供了兩種後處理 bean 的擴展接⼝，分別為 BeanPostProcessor 和 BeanFactoryPostProcessor，兩者在使⽤上是有所區別的。

⼯⼚初始化（BeanFactory）—> Bean 對象

在 BeanFactory 初始化之後可以使⽤ BeanFactoryPostProcessor 進⾏後置處理做⼀些事情

在 Bean 對象實例化（並不是 Bean 的整個⽣命週期完成）之後可以使⽤ BeanPostProcessor 進⾏後置處理做⼀些事情

注意：對像不⼀定是 springbean，⽽ springbean ⼀定是個對象

SpringBean 的⽣命週期

#### 2.3.1 BeanPostProcessor
BeanPostProcessor 是針對 Bean 級別的處理，可以針對某個具體的 Bean.
![[Pasted image 20211003141933.png]]

該接⼝提供了兩個⽅法，分別在 Bean 的初始化⽅法前和初始化⽅法後執⾏，具體這個初始化⽅法指的是什麼⽅法，類似我們在定義 bean 時，定義了 init-method 所指定的⽅法

定義⼀個類實現了 BeanPostProcessor，默認是會對整個 Spring 容器中所有的 bean 進⾏處理。如果要對具體的某個 bean 處理，可以通過⽅法參數判斷，兩個類型參數分別為 Object 和 String，

第⼀個參數是每個 bean 的實例，第⼆個參數是每個 bean 的 name 或者 id 屬性的值。所以我們可以通過第⼆個參數，來判斷我們將要處理的具體的 bean。

注意：處理是發⽣在 Spring 容器的實例化和依賴注⼊之後。

#### 2.3.2 BeanFactoryPostProcessor
BeanFactory 級別的處理，是針對整個 Bean 的⼯⼚進⾏處理，典型應⽤: PropertyPlaceholderConfigurer
![[Pasted image 20211003142117.png]]

此接⼝只提供了⼀個⽅法，⽅法參數為 ConfigurableListableBeanFactory，該參數類型定義了⼀些⽅法
![[Pasted image 20211003142147.png]]

其中有個⽅法名為 getBeanDefinition 的⽅法，我們可以根據此⽅法，找到我們定義 bean 的 BeanDefinition 對象。然後我們可以對定義的屬性進⾏修改，以下是 BeanDefinition 中的⽅法
![[Pasted image 20211003142226.png]]

⽅法名字類似我們 bean 標籤的屬性，setBeanClassName 對應 bean 標籤中的 class 屬性，所以當我們拿到 BeanDefinition 對象時，我們可以⼿動修改 bean 標籤中所定義的屬性值。

BeanDefinition 對象：我們在 XML 中定義的 bean 標籤，Spring 解析 bean 標籤成為⼀個 JavaBean，這個 JavaBean 就是 BeanDefinition

注意：調⽤ BeanFactoryPostProcessor ⽅法時，這時候 bean 還沒有實例化，此時 bean 剛被解析成 BeanDefinition 對象