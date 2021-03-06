---
date: 2021-11-24
aliases: []
---

# Metadata

**Title** :: Spring

**Author** :: #Matthew 

**Classification** :: #Learn #Java #Basic #Spring

**Status** :: #ð±

**Type** :: #Note

**Previous** ::[[Study_Matthew/Spring/2_ç¬¬äºé¨å æ ¸å¿ææ³]]

**ParentNode** :: [[Study_Matthew/Spring/Spring|Spring]]

---


# ç¬¬ä¸é¨å æå¯«å¯¦ç¾ IoC å AOP

ä¸â¼é¨åæåçè§£äº IoC å AOP ææ³ï¼æååä¸èæ® Spring æ¯å¦ä½å¯¦ç¾éå©åææ³çï¼æ­¤èæºåäºâ¼åãéâ¾è½è³¬ãçæ¡ä¾ï¼è«åæè©²æ¡ä¾å¨ä»£ç¢¼å±¤æ¬¡æä»éº¼åé¡ ï¼åæä¹å¾ä½¿â½¤æåå·²æç¥è­è§£æ±ºéäºåé¡ï¼çé»ï¼ãå¶å¯¦éåéç¨æåå°±æ¯å¨â¼æ­¥æ­¥åæä¸¦â¼¿å¯«å¯¦ç¾ IoC å AOPã

## ç¬¬ 1 ç¯ éè¡è½å¸³æ¡ä¾ä»é¢

![[Spring ç¬¬ä¸é¨å -  éè¡è½å¸³æ¡ä¾ä»é¢.png]]

## ç¬¬ 2 ç¯ éè¡è½å¸³æ¡ä¾è¡¨çµæ§

![[Spring ç¬¬ä¸é¨å -  éè¡è½å¸³æ¡ä¾è¡¨çµæ§.png]]

## ç¬¬ 3 ç¯ éè¡è½å¸³æ¡ä¾ä»£ç¢¼èª¿ç¨éä¿

![[Spring ç¬¬ä¸é¨å -  éè¡è½å¸³æ¡ä¾ä»£ç¢¼èª¿ç¨éä¿.png]]

## ç¬¬ 4 ç¯ éè¡è½å¸³æ¡ä¾ééµä»£ç¢¼

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
    * @author åºç«  
    */  
    @WebServlet(name="transferServlet",urlPatterns = "/transferServlet")  
    public class TransferServlet extends HttpServlet {  
     // 1. å®ä¾å service å±å¯¹è±¡  
     private TransferService transferService = new TransferServiceImpl();  
     @Override  
     protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
    throws ServletException, IOException {  
     doPost(req,resp);  
     }  
     @Override  
     protected void doPost(HttpServletRequest req, HttpServletResponse  
    resp) throws ServletException, IOException {  
     // è®¾ç½®è¯·æ±ä½çå­ç¬¦ç¼ç   
     req.setCharacterEncoding("UTF-8");  
     String fromCardNo = req.getParameter("fromCardNo");  
     String toCardNo = req.getParameter("toCardNo");  
     String moneyStr = req.getParameter("money");  
     int money = Integer.parseInt(moneyStr);  
     Result result = new Result();  
     try {  
     // 2. è°â½¤ service å±â½æ³  
     transferService.transfer(fromCardNo,toCardNo,money);  
     result.setStatus("200");  
     } catch (Exception e) {  
     e.printStackTrace();  
     result.setStatus("201");  
     result.setMessage(e.toString());  
     }  
     // ååº  
     resp.setContentType("application/json;charset=utf-8");  
     resp.getWriter().print(JsonUtils.object2Json(result));  
     }  
    }
```
    
-   TransferService æ¥å£åå¯¦ç¾é¡
```java
    package com.lagou.edu.service;  
    /**  
    * @author åºç«  
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
    * @author åºç«  
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
-   AccountDao å±¤æ¥å£ååºæ¼ JDBC çå¯¦ç¾é¡
```java    
    package com.lagou.edu.dao;  
    import com.lagou.edu.pojo.Account;  
    /**  
    * @author åºç«  
    */  
    public interface AccountDao {  
     Account queryAccountByCardNo(String cardNo) throws Exception;  
     int updateAccountByCardNo(Account account) throws Exception;  
    }
```    
-   JdbcAccountDaoImplï¼Jdbc æè¡å¯¦ç¾ Dao å±¤æ¥å£ï¼
```java    
    package com.lagou.edu.dao.impl;  
    import com.lagou.edu.pojo.Account;  
    import com.lagou.edu.dao.AccountDao;  
    import com.lagou.edu.utils.DruidUtils;  
    import java.sql.Connection;  
    import java.sql.PreparedStatement;  
    import java.sql.ResultSet;  
    /**  
    * @author åºç«  
    */  
    public class JdbcAccountDaoImpl implements AccountDao {  
     @Override  
     public Account queryAccountByCardNo(String cardNo) throws Exception {  
     //ä»è¿æ¥æ± è·åè¿æ¥  
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
    â  
     @Override  
     public int updateAccountByCardNo(Account account) throws Exception {  
			 //ä»è¿æ¥æ± è·åè¿æ¥  
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


## ç¬¬ 5 ç¯ éè¡è½å¸³æ¡ä¾ä»£ç¢¼åé¡åæ

![[Spring ç¬¬ä¸é¨å -  éè¡è½å¸³æ¡ä¾ä»£ç¢¼åé¡åæ.png]]

ï¼1ï¼åé¡â¼ï¼å¨ä¸è¿°æ¡ä¾å¯¦ç¾ä¸­ï¼service å±¤å¯¦ç¾é¡å¨ä½¿â½¤ dao å±¤å°è±¡æï¼ç´æ¥å¨ TransferServiceImpl ä¸­éé AccountDao accountDao = new JdbcAccountDaoImpl() ç²å¾äº dao å±¤å° è±¡ï¼ç¶â½½â¼å new ééµå­å»å° TransferServiceImpl å dao å±¤å·é«çâ¼åå¯¦ç¾é¡ JdbcAccountDaoImpl è¦åå¨äºâ¼èµ·ï¼å¦æèªªæè¡æ¶æ§ç¼â½£â¼äºè®åï¼dao å±¤çå¯¦ç¾è¦ä½¿â½¤å¶å®æè¡ï¼â½å¦ Mybatisï¼æèåæèµ·ä¾çææ¬ï¼æ¯â¼å new çå°â½é½éè¦ä¿®æ¹æºä»£ç¢¼ï¼éæ°ç·¨è­¯ï¼â¾¯åæ¥â¼éç¼çæç¾©å°â¼¤æææ£ï¼

ï¼2ï¼åé¡â¼ï¼service å±¤ä»£ç¢¼æ²æç«ç¶éæ²æé²â¾äºåæ§å¶ ï¼ ï¼å¦æè½è³¬éç¨ä¸­åºç¾ç°å¸¸ï¼å°å¯è½å°è´æ¸æåº«æ¸æé¯äºï¼å¾æå¯è½æå¾å´éï¼å°¤å¶å¨â¾¦èæ¥­åã


## ç¬¬ 6 ç¯ åé¡è§£æ±ºæè·¯
- éå°åé¡â¼æèï¼
	- å¯¦ä¾åå°è±¡çâ½å¼é¤äº new ä¹å¤ï¼éæä»éº¼æè¡ï¼åå° (éè¦æé¡çå¨éå®é¡åéç½®å¨ xml ä¸­)
- èæ®ä½¿â½¤è¨­è¨æ¨¡å¼ä¸­çâ¼¯â¼æ¨¡å¼è§£è¦åï¼å¦å¤é â½¬ä¸­å¾å¾æå¾å¤å°è±¡éè¦å¯¦ä¾åï¼é£å°±å¨â¼¯â¼ä¸­ä½¿â½¤å å°æè¡å¯¦ä¾åå°è±¡ï¼â¼¯â¼æ¨¡å¼å¾åé©
![[Pasted image 20210909141842.png]]

- æ´é²â¼æ­¥ï¼ä»£ç¢¼ä¸­è½å¦åªè²ææéå¯¦ä¾çæ¥â¼é¡åï¼ä¸åºç¾ new ä¹ä¸åºç¾â¼¯â¼é¡çå­ç¼ï¼å¦ä¸åï¼è½ï¼è²æâ¼åè®éä¸¦æä¾ set â½æ³ï¼å¨åå°çæåå°æéè¦çå°è±¡æ³¨â¼é²å»å§
![[Pasted image 20210909141429.png]]

- éå°åé¡â¼æèï¼
	- service å±¤æ²ææ·»å äºåæ§å¶ï¼æéº¼è¾¦ï¼æ²æäºåå°±æ·»å ä¸äºåæ§å¶ï¼â¼¿åæ§å¶ JDBC ç Connection äºåï¼ä½è¦æ³¨æå° Connection åç¶åç·ç¨ç¶å®ï¼å³ä¿è­â¼åç·ç¨åªæâ¼å Connectionï¼éæ¨£æä½æéå°çæ¯åâ¼å Connectionï¼é²â½½æ§å¶çæ¯åâ¼åäºåï¼
![[Pasted image 20210909141559.png]]

## ç¬¬ 7 ç¯ æ¡ä¾ä»£ç¢¼æ¹é 

ï¼1ï¼éå°åé¡ä¸çä»£ç¢¼æ¹é 

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

- å¢å  BeanFactory.java
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
	* @author åºç«
	*/
	public class BeanFactory {
	 /**
	 * â¼¯â¼ç±»çä¸¤ä¸ªä»»å¡
	 * ä»»å¡â¼ï¼å è½½è§£æ xmlï¼è¯»å xml ä¸­ç bean ä¿¡æ¯ï¼éè¿åå°ææ¯å®ä¾å bean å¯¹è±¡ï¼ç¶åæ¾â¼ map å¾â½¤
	 * ä»»å¡â¼ï¼æä¾æ¥â¼â½æ³æ ¹æ® id ä» map ä¸­è·å beanï¼éæâ½æ³ï¼
	 */
	 private static Map<String,Object> map = new HashMap<>();
	 static {
		 InputStream resourceAsStream = BeanFactory.class.getClassLoader().getResourceAsStream("beans.xml");
		 SAXReader saxReader = new SAXReader();
		 try {
			 Document document = saxReader.read(resourceAsStream);
			 Element rootElement = document.getRootElement();
			 List<Element> list = rootElement.selectNodes("//bean");
			 // å®ä¾å bean å¯¹è±¡
			 for (int i = 0; i < list.size(); i++) {
				 Element element = list.get(i);
				 String id = element.attributeValue("id");
				 String clazz = element.attributeValue("class");
				 Class<?> aClass = Class.forName(clazz);
				 Object o = aClass.newInstance();
				 map.put(id,o);
			 }
			 // ç»´æ¤ bean ä¹é´çä¾èµå³ç³»
			 List<Element> propertyNodes = rootElement.selectNodes("//property");
			 for (int i = 0; i < propertyNodes.size(); i++) {
				 Element element = propertyNodes.get(i);
				 // å¤ç property åç´ 
				 String name = element.attributeValue("name");
				 String ref = element.attributeValue("ref");

				 String parentId = element.getParent().attributeValue("id");
				 Object parentObject = map.get(parentId);
				 Method[] methods = parentObject.getClass().getMethods();
				 for (int j = 0; j < methods.length; j++) {
					 Method method = methods[j];
					 if(("set" + name).equalsIgnoreCase(method.getName()))
					 {
						 // bean ä¹é´çä¾èµå³ç³»ï¼æ³¨â¼ beanï¼
						Object propertyObject = map.get(ref);
						 method.invoke(parentObject,propertyObject);
					 }
				 }
				 // ç»´æ¤ä¾èµå³ç³»åéæ°å° bean æ¾â¼ map ä¸­
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

- ä¿®æ¹ TransferServlet
![[Pasted image 20210909143233.png]]

- ä¿®æ¹ TransferServiceImpl
![[Pasted image 20210909143311.png]]

ï¼2ï¼éå°åé¡äºçæ¹é 
- å¢å  ConnectionUtils
	```java
	package com.lagou.edu.utils;
	import java.sql.Connection;
	import java.sql.SQLException;
	/**
	* @author åºç«
	*/
	public class ConnectionUtils {
	 /*private ConnectionUtils() {
	 }
	 private static ConnectionUtils connectionUtils = new
	ConnectionUtils();
	 public static ConnectionUtils getInstance() {
	 return connectionUtils;
	 }*/
	 private ThreadLocal<Connection> threadLocal = new ThreadLocal<>(); //å­å¨å½åçº¿ç¨çè¿æ¥
	 /**
	 * ä»å½åçº¿ç¨è·åè¿æ¥
	 */
	 public Connection getCurrentThreadConn() throws SQLException {
	 /**
	 * å¤æ­å½åçº¿ç¨ä¸­æ¯å¦å·²ç»ç»å®è¿æ¥ï¼å¦ææ²¡æç»å®ï¼éè¦ä»è¿æ¥æ± è·åâ¼ä¸ªè¿æ¥ç»å®å°
	å½åçº¿ç¨
	 */
	 Connection connection = threadLocal.get();
	 if(connection == null) {
	 // ä»è¿æ¥æ± æ¿è¿æ¥å¹¶ç»å®å°çº¿ç¨
	 connection = DruidUtils.getInstance().getConnection();
	 // ç»å®å°å½åçº¿ç¨
	 threadLocal.set(connection);
	 }
	 return connection;
	 }
	}
	```

- å¢å  TransactionManager äºåç®¡çå¨é¡
	```java
	package com.lagou.edu.utils;
	import java.sql.SQLException;
	/**
	* @author åºç«
	*/
	public class TransactionManager {
	 private ConnectionUtils connectionUtils;
	 public void setConnectionUtils(ConnectionUtils connectionUtils) {
	 this.connectionUtils = connectionUtils;
	 }
	 // å¼å¯äºå¡
	 public void beginTransaction() throws SQLException {
	 connectionUtils.getCurrentThreadConn().setAutoCommit(false);
	 }
	 // æäº¤äºå¡
	 public void commit() throws SQLException {
	 connectionUtils.getCurrentThreadConn().commit();
	 }
	 // åæ»äºå¡
	 public void rollback() throws SQLException {
	 connectionUtils.getCurrentThreadConn().rollback();
	 }
	}
	```

- å¢å  ProxyFactory ä»£çå·¥å» é¡
	```java
	package com.lagou.edu.factory;
	import com.lagou.edu.utils.TransactionManager;
	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	import java.lang.reflect.Proxy;
	/**
	* @author åºç«
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
						 // å¼å¯äºå¡
						 transactionManager.beginTransaction();
						 // è°â½¤åæä¸å¡é»è¾
						 result = method.invoke(target,args);
						 // æäº¤äºå¡
						 transactionManager.commit();
					 }catch(Exception e) {
						 e.printStackTrace();
						 // åæ»äºå¡
						 transactionManager.rollback();
						 // å¼å¸¸åä¸æåº,ä¾¿äº servlet ä¸­æè·
						 throw e.getCause();
					 }
						return result;
				 }
			});
		}
	}
	```

- ä¿®æ¹ beans.xml
	```xml
	<?xml version="1.0" encoding="UTF-8" ?>
	<!--è·æ ç­¾ beansï¼â¾¥â¾¯éç½®â¼ä¸ªâ¼â¼ä¸ªç bean â¼¦æ ç­¾ï¼æ¯â¼ä¸ª bean â¼¦æ ç­¾é½ä»£è¡¨â¼ä¸ªç±»çéç½®--
	>
	<beans>
	 <!--id æ è¯å¯¹è±¡ï¼class æ¯ç±»çå¨éå®ç±»å-->
	 <bean id="accountDao"
	class="com.lagou.edu.dao.impl.JdbcAccountDaoImpl">
	 <property name="ConnectionUtils" ref="connectionUtils"/>
	 </bean>
	 <bean id="transferService"
	class="com.lagou.edu.service.impl.TransferServiceImpl">
	 <!--set+ name ä¹åéå®å°ä¼ å¼ç set â½æ³äºï¼éè¿åå°ææ¯å¯ä»¥è°â½¤è¯¥â½æ³ä¼ â¼å¯¹åº
	çå¼-->
	 <property name="AccountDao" ref="accountDao"></property>
	 </bean>
	 <!--éç½®æ°å¢çä¸ä¸ª Bean-->
	 <bean id="connectionUtils"
	class="com.lagou.edu.utils.ConnectionUtils"></bean>
	 <!--äºå¡ç®¡çå¨-->
	 <bean id="transactionManager"
	class="com.lagou.edu.utils.TransactionManager">
	 <property name="ConnectionUtils" ref="connectionUtils"/>
	 </bean>
	 <!--ä»£çå¯¹è±¡â¼¯â¼-->
	 <bean id="proxyFactory" class="com.lagou.edu.factory.ProxyFactory">
	 <property name="TransactionManager" ref="transactionManager"/>
	 </bean>
	</beans>
	```

- ä¿®æ¹ jdbcAccountDaoImpl
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
	* @author åºç«
	*/
	public class JdbcAccountDaoImpl implements AccountDao {
	 private ConnectionUtils connectionUtils;
	 public void setConnectionUtils(ConnectionUtils connectionUtils) {
		this.connectionUtils = connectionUtils;
	 }

	 @Override
	 public Account queryAccountByCardNo(String cardNo) throws Exception {
		 //ä»è¿æ¥æ± è·åè¿æ¥
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
		 // ä»è¿æ¥æ± è·åè¿æ¥
		 // æ¹é ä¸ºï¼ä»å½åçº¿ç¨å½ä¸­è·åç»å®ç connection è¿æ¥
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
