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
# 第三部分 手寫實現 IoC 和 AOP

上⼀部分我們理解了 IoC 和 AOP 思想，我們先不考慮 Spring 是如何實現這兩個思想的，此處準備了⼀個『銀⾏轉賬』的案例，請分析該案例在代碼層次有什麼問題 ？分析之後使⽤我們已有知識解決這些問題（痛點）。其實這個過程我們就是在⼀步步分析並⼿寫實現 IoC 和 AOP。

## 第 1 節 銀行轉帳案例介面

![[Spring 第三部分 -  銀行轉帳案例介面.png]]

## 第 2 節 銀行轉帳案例表結構

![[Spring 第三部分 -  銀行轉帳案例表結構.png]]

## 第 3 節 銀行轉帳案例代碼調用關係

![[Spring 第三部分 -  銀行轉帳案例代碼調用關係.png]]

## 第 4 節 銀行轉帳案例關鍵代碼

```java
-   TransferServlet
    
    package com.lagou.edu.servlet;  
    import com.lagou.edu.service.impl.TransferServiceImpl;  
    import com.lagou.edu.utils.JsonUtils;  
    import com.lagou.edu.pojo.Result;  
    import com.lagou.edu.service.TransferService;  
    import javax.servlet.ServletException;  
    import javax.servlet.annotation.WebServlet;  
    import javax.servlet.http.HttpServlet;  
    import javax.servlet.http.HttpServletRequest;  
    import javax.servlet.http.HttpServletResponse;  
    import java.io.IOException;  
    /**  
    * @author 应癫  
    */  
    @WebServlet(name="transferServlet",urlPatterns = "/transferServlet")  
    public class TransferServlet extends HttpServlet {  
     // 1. 实例化service层对象  
     private TransferService transferService = new TransferServiceImpl();  
     @Override  
     protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
    throws ServletException, IOException {  
     doPost(req,resp);  
     }  
     @Override  
     protected void doPost(HttpServletRequest req, HttpServletResponse  
    resp) throws ServletException, IOException {  
     // 设置请求体的字符编码  
     req.setCharacterEncoding("UTF-8");  
     String fromCardNo = req.getParameter("fromCardNo");  
     String toCardNo = req.getParameter("toCardNo");  
     String moneyStr = req.getParameter("money");  
     int money = Integer.parseInt(moneyStr);  
     Result result = new Result();  
     try {  
     // 2. 调⽤service层⽅法  
     transferService.transfer(fromCardNo,toCardNo,money);  
     result.setStatus("200");  
     } catch (Exception e) {  
     e.printStackTrace();  
     result.setStatus("201");  
     result.setMessage(e.toString());  
     }  
     // 响应  
     resp.setContentType("application/json;charset=utf-8");  
     resp.getWriter().print(JsonUtils.object2Json(result));  
     }  
    }
```
    
-   TransferService 接口及實現類
```java
    package com.lagou.edu.service;  
    /**  
    * @author 应癫  
    */  
    public interface TransferService {  
     void transfer(String fromCardNo,String toCardNo,int money) throws Exception;  
    }
    
    package com.lagou.edu.service.impl;  
    import com.lagou.edu.dao.AccountDao;  
    import com.lagou.edu.dao.impl.JdbcAccountDaoImpl;  
    import com.lagou.edu.pojo.Account;  
    import com.lagou.edu.service.TransferService;  
    /**  
    * @author 应癫  
    */  
    public class TransferServiceImpl implements TransferService {  
     private AccountDao accountDao = new JdbcAccountDaoImpl();  
     @Override  
     public void transfer(String fromCardNo, String toCardNo, int money) throws Exception {  
			 Account from = accountDao.queryAccountByCardNo(fromCardNo);  
			 Account to = accountDao.queryAccountByCardNo(toCardNo);  
			 from.setMoney(from.getMoney()-money);  
			 to.setMoney(to.getMoney()+money);  
			 accountDao.updateAccountByCardNo(from);  
			 accountDao.updateAccountByCardNo(to);  
     }  
    }
```    
-   AccountDao 層接口及基於JDBC的實現類
```java    
    package com.lagou.edu.dao;  
    import com.lagou.edu.pojo.Account;  
    /**  
    * @author 应癫  
    */  
    public interface AccountDao {  
     Account queryAccountByCardNo(String cardNo) throws Exception;  
     int updateAccountByCardNo(Account account) throws Exception;  
    }
```    
-   JdbcAccountDaoImpl（Jdbc技術實現Dao層接口）
```java    
    package com.lagou.edu.dao.impl;  
    import com.lagou.edu.pojo.Account;  
    import com.lagou.edu.dao.AccountDao;  
    import com.lagou.edu.utils.DruidUtils;  
    import java.sql.Connection;  
    import java.sql.PreparedStatement;  
    import java.sql.ResultSet;  
    /**  
    * @author 应癫  
    */  
    public class JdbcAccountDaoImpl implements AccountDao {  
     @Override  
     public Account queryAccountByCardNo(String cardNo) throws Exception {  
     //从连接池获取连接  
			 Connection con = DruidUtils.getInstance().getConnection();  
			 String sql = "select * from account where cardNo=?";  
			 PreparedStatement preparedStatement = con.prepareStatement(sql);  
			 preparedStatement.setString(1,cardNo);  
			 ResultSet resultSet = preparedStatement.executeQuery();  
			 Account account = new Account();  
			 while(resultSet.next()) {  
				 account.setCardNo(resultSet.getString("cardNo"));  
				 account.setName(resultSet.getString("name"));  
				 account.setMoney(resultSet.getInt("money"));  
			 }  
			 resultSet.close();  
			 preparedStatement.close();  
			 con.close();  
			 return account;  
     }  
    ​  
     @Override  
     public int updateAccountByCardNo(Account account) throws Exception {  
			 //从连接池获取连接  
			 Connection con = DruidUtils.getInstance().getConnection();  
			 String sql = "update account set money=? where cardNo=?";  
			 PreparedStatement preparedStatement = con.prepareStatement(sql);  
			 preparedStatement.setInt(1,account.getMoney());  
			 preparedStatement.setString(2,account.getCardNo());  
			 int i = preparedStatement.executeUpdate();  
			 preparedStatement.close();  
			 con.close();  
     return i;  
     }  
    }
```


## 第 5 節 銀行轉帳案例代碼問題分析

![[Spring 第三部分 -  銀行轉帳案例代碼問題分析.png]]

（1）問題⼀：在上述案例實現中，service 層實現類在使⽤ dao 層對象時，直接在 TransferServiceImpl 中通過 AccountDao accountDao = new JdbcAccountDaoImpl() 獲得了 dao層對 象，然⽽⼀個 new 關鍵字卻將 TransferServiceImpl 和 dao 層具體的⼀個實現類 JdbcAccountDaoImpl 耦合在了⼀起，如果說技術架構發⽣⼀些變動，dao 層的實現要使⽤其它技術，⽐如 Mybatis，思考切換起來的成本？每⼀個 new 的地⽅都需要修改源代碼，重新編譯，⾯向接⼝開發的意義將⼤打折扣？

（2）問題⼆：service 層代碼沒有竟然還沒有進⾏事務控制 ？ ！如果轉賬過程中出現異常，將可能導致數據庫數據錯亂，後果可能會很嚴重，尤其在⾦融業務。


## 第 6 節 問題解決思路
- 針對問題⼀思考：
	- 實例化對象的⽅式除了 new 之外，還有什麼技術？反射 (需要把類的全限定類名配置在xml中)
- 考慮使⽤設計模式中的⼯⼚模式解耦合，另外項⽬中往往有很多對象需要實例化，那就在⼯⼚中使⽤反 射技術實例化對象，⼯⼚模式很合適
![[Pasted image 20210909141842.png]]

- 更進⼀步，代碼中能否只聲明所需實例的接⼝類型，不出現 new 也不出現⼯⼚類的字眼，如下圖？能！聲明⼀個變量並提供 set ⽅法，在反射的時候將所需要的對象注⼊進去吧
![[Pasted image 20210909141429.png]]

- 針對問題⼆思考：
	- service 層沒有添加事務控制，怎麼辦？沒有事務就添加上事務控制，⼿動控制 JDBC 的 Connection 事務，但要注意將Connection和當前線程綁定（即保證⼀個線程只有⼀個Connection，這樣操作才針對的是同⼀個 Connection，進⽽控制的是同⼀個事務）
![[Pasted image 20210909141559.png]]

## 第 7 節 案例代碼改造

（1）針對問題一的代碼改造

- beans.xml
	```xml
	<?xml version="1.0" encoding="UTF-8" ?>
	<beans>
	 <bean id="transferService"
	class="com.lagou.edu.service.impl.TransferServiceImpl">
	 <property name="AccountDao" ref="accountDao"></property>
	 </bean>
	 <bean id="accountDao"
	class="com.lagou.edu.dao.impl.JdbcAccountDaoImpl">
	 </bean>
	</beans>
	```

- 增加 BeanFactory.java
	```java
	package com.lagou.edu.factory;
	import org.dom4j.Document;
	import org.dom4j.DocumentException;
	import org.dom4j.Element;
	import org.dom4j.io.SAXReader;
	import java.io.InputStream;
	import java.lang.reflect.InvocationTargetException;
	import java.lang.reflect.Method;
	import java.util.HashMap;
	import java.util.List;
	import java.util.Map;
	/**
	* @author 应癫
	*/
	public class BeanFactory {
	 /**
	 * ⼯⼚类的两个任务
	 * 任务⼀：加载解析xml，读取xml中的bean信息，通过反射技术实例化bean对象，然后放⼊map待⽤
	 * 任务⼆：提供接⼝⽅法根据id从map中获取bean（静态⽅法）
	 */
	 private static Map<String,Object> map = new HashMap<>();
	 static {
		 InputStream resourceAsStream = BeanFactory.class.getClassLoader().getResourceAsStream("beans.xml");
		 SAXReader saxReader = new SAXReader();
		 try {
			 Document document = saxReader.read(resourceAsStream);
			 Element rootElement = document.getRootElement();
			 List<Element> list = rootElement.selectNodes("//bean");
			 // 实例化bean对象
			 for (int i = 0; i < list.size(); i++) {
				 Element element = list.get(i);
				 String id = element.attributeValue("id");
				 String clazz = element.attributeValue("class");
				 Class<?> aClass = Class.forName(clazz);
				 Object o = aClass.newInstance();
				 map.put(id,o);
			 }
			 // 维护bean之间的依赖关系
			 List<Element> propertyNodes = rootElement.selectNodes("//property");
			 for (int i = 0; i < propertyNodes.size(); i++) {
				 Element element = propertyNodes.get(i);
				 // 处理property元素
				 String name = element.attributeValue("name");
				 String ref = element.attributeValue("ref");

				 String parentId = element.getParent().attributeValue("id");
				 Object parentObject = map.get(parentId);
				 Method[] methods = parentObject.getClass().getMethods();
				 for (int j = 0; j < methods.length; j++) {
					 Method method = methods[j];
					 if(("set" + name).equalsIgnoreCase(method.getName()))
					 {
						 // bean之间的依赖关系（注⼊bean）
						Object propertyObject = map.get(ref);
						 method.invoke(parentObject,propertyObject);
					 }
				 }
				 // 维护依赖关系后重新将bean放⼊map中
				 map.put(parentId,parentObject);
			 }
		 } catch (DocumentException e) {
		 e.printStackTrace();
		 } catch (ClassNotFoundException e) {
		 e.printStackTrace();
		 } catch (IllegalAccessException e) {
		 e.printStackTrace();
		 } catch (InstantiationException e) {
		 e.printStackTrace();
		 } catch (InvocationTargetException e) {
		 e.printStackTrace();
		 }
	 }
	 public static Object getBean(String id) {
	 return map.get(id);
	 }
	}
	```

- 修改 TransferServlet
![[Pasted image 20210909143233.png]]

- 修改 TransferServiceImpl
![[Pasted image 20210909143311.png]]

（2）針對問題二的改造
- 增加 ConnectionUtils
	```java
	package com.lagou.edu.utils;
	import java.sql.Connection;
	import java.sql.SQLException;
	/**
	* @author 应癫
	*/
	public class ConnectionUtils {
	 /*private ConnectionUtils() {
	 }
	 private static ConnectionUtils connectionUtils = new
	ConnectionUtils();
	 public static ConnectionUtils getInstance() {
	 return connectionUtils;
	 }*/
	 private ThreadLocal<Connection> threadLocal = new ThreadLocal<>(); //存储当前线程的连接
	 /**
	 * 从当前线程获取连接
	 */
	 public Connection getCurrentThreadConn() throws SQLException {
	 /**
	 * 判断当前线程中是否已经绑定连接，如果没有绑定，需要从连接池获取⼀个连接绑定到
	当前线程
	 */
	 Connection connection = threadLocal.get();
	 if(connection == null) {
	 // 从连接池拿连接并绑定到线程
	 connection = DruidUtils.getInstance().getConnection();
	 // 绑定到当前线程
	 threadLocal.set(connection);
	 }
	 return connection;
	 }
	}
	```

- 增加 TransactionManager 事務管理器類
	```java
	package com.lagou.edu.utils;
	import java.sql.SQLException;
	/**
	* @author 应癫
	*/
	public class TransactionManager {
	 private ConnectionUtils connectionUtils;
	 public void setConnectionUtils(ConnectionUtils connectionUtils) {
	 this.connectionUtils = connectionUtils;
	 }
	 // 开启事务
	 public void beginTransaction() throws SQLException {
	 connectionUtils.getCurrentThreadConn().setAutoCommit(false);
	 }
	 // 提交事务
	 public void commit() throws SQLException {
	 connectionUtils.getCurrentThreadConn().commit();
	 }
	 // 回滚事务
	 public void rollback() throws SQLException {
	 connectionUtils.getCurrentThreadConn().rollback();
	 }
	}
	```

- 增加 ProxyFactory 代理工廠類
	```java
	package com.lagou.edu.factory;
	import com.lagou.edu.utils.TransactionManager;
	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	import java.lang.reflect.Proxy;
	/**
	* @author 应癫
	*/
	public class ProxyFactory {
	 private TransactionManager transactionManager;
	 public void setTransactionManager(TransactionManager transactionManager) {
		this.transactionManager = transactionManager;
	 }
	 public Object getProxy(Object target) {
		return Proxy.newProxyInstance(
			this.getClass().getClassLoader(), 
			target.getClass().getInterfaces(), 
			new InvocationHandler() {

				 @Override
				 public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
					 Object result = null;
					 try{
						 // 开启事务
						 transactionManager.beginTransaction();
						 // 调⽤原有业务逻辑
						 result = method.invoke(target,args);
						 // 提交事务
						 transactionManager.commit();
					 }catch(Exception e) {
						 e.printStackTrace();
						 // 回滚事务
						 transactionManager.rollback();
						 // 异常向上抛出,便于servlet中捕获
						 throw e.getCause();
					 }
						return result;
				 }
			});
		}
	}
	```

- 修改 beans.xml
	```xml
	<?xml version="1.0" encoding="UTF-8" ?>
	<!--跟标签beans，⾥⾯配置⼀个⼜⼀个的bean⼦标签，每⼀个bean⼦标签都代表⼀个类的配置--
	>
	<beans>
	 <!--id标识对象，class是类的全限定类名-->
	 <bean id="accountDao"
	class="com.lagou.edu.dao.impl.JdbcAccountDaoImpl">
	 <property name="ConnectionUtils" ref="connectionUtils"/>
	 </bean>
	 <bean id="transferService"
	class="com.lagou.edu.service.impl.TransferServiceImpl">
	 <!--set+ name 之后锁定到传值的set⽅法了，通过反射技术可以调⽤该⽅法传⼊对应
	的值-->
	 <property name="AccountDao" ref="accountDao"></property>
	 </bean>
	 <!--配置新增的三个Bean-->
	 <bean id="connectionUtils"
	class="com.lagou.edu.utils.ConnectionUtils"></bean>
	 <!--事务管理器-->
	 <bean id="transactionManager"
	class="com.lagou.edu.utils.TransactionManager">
	 <property name="ConnectionUtils" ref="connectionUtils"/>
	 </bean>
	 <!--代理对象⼯⼚-->
	 <bean id="proxyFactory" class="com.lagou.edu.factory.ProxyFactory">
	 <property name="TransactionManager" ref="transactionManager"/>
	 </bean>
	</beans>
	```

- 修改 jdbcAccountDaoImpl
	```java
	package com.lagou.edu.dao.impl;
	import com.lagou.edu.pojo.Account;
	import com.lagou.edu.dao.AccountDao;
	import com.lagou.edu.utils.ConnectionUtils;
	import com.lagou.edu.utils.DruidUtils;
	import java.sql.Connection;
	import java.sql.PreparedStatement;
	import java.sql.ResultSet;
	/**
	* @author 应癫
	*/
	public class JdbcAccountDaoImpl implements AccountDao {
	 private ConnectionUtils connectionUtils;
	 public void setConnectionUtils(ConnectionUtils connectionUtils) {
		this.connectionUtils = connectionUtils;
	 }

	 @Override
	 public Account queryAccountByCardNo(String cardNo) throws Exception {
		 //从连接池获取连接
		 // Connection con = DruidUtils.getInstance().getConnection();
		 Connection con = connectionUtils.getCurrentThreadConn();
		 String sql = "select * from account where cardNo=?";
		 PreparedStatement preparedStatement = con.prepareStatement(sql);
		 preparedStatement.setString(1,cardNo);
		 ResultSet resultSet = preparedStatement.executeQuery();
		 Account account = new Account();
		 while(resultSet.next()) {
			 account.setCardNo(resultSet.getString("cardNo"));
			 account.setName(resultSet.getString("name"));
			 account.setMoney(resultSet.getInt("money"));
		 }
		 resultSet.close();
		 preparedStatement.close();
		 //con.close();
		 return account;
	 }

	 @Override
	 public int updateAccountByCardNo(Account account) throws Exception {
		 // 从连接池获取连接
		 // 改造为：从当前线程当中获取绑定的connection连接
		 //Connection con = DruidUtils.getInstance().getConnection();
		 Connection con = connectionUtils.getCurrentThreadConn();
		 String sql = "update account set money=? where cardNo=?";
		 PreparedStatement preparedStatement = con.prepareStatement(sql);
		 preparedStatement.setInt(1,account.getMoney());
		 preparedStatement.setString(2,account.getCardNo());
		 int i = preparedStatement.executeUpdate();
		 preparedStatement.close();
		 //con.close();
		 return i;
	 }

	}
	```
