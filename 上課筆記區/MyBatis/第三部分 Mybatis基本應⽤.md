---
界: Framework
門: MyBatis
綱: 
tags: ["#MyBatis","#BQ"]
aliases:
  - 
date: {{DATE:YYYY-MM-DD}}
---

# 第三部分：Mybatis基本应⽤

## 3.1 快速入門
MyBatis 官網地址：[MyBatis 官網]http://www.mybatis.org/mybatis-3/

![[Pasted image 20210927114618.png | 500]]

### 3.1.0 開發步驟
①  添加MyBatis的坐標

②  創建user數據表

③  編寫User實體類

④  編寫映射⽂件UserMapper.xml

⑤  編寫核⼼⽂件SqlMapConfig.xml

⑥  編寫測試類

### 3.1.1 環境搭建
1. 導⼊MyBatis的坐標和其他相關坐標
```xml
<properties>
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
<maven.compiler.encoding>UTF-8</maven.compiler.encoding>
<java.version>1.8</java.version>
<maven.compiler.source>1.8</maven.compiler.source>
<maven.compiler.target>1.8</maven.compiler.target>
</properties>
<!--mybatis坐标-->
<dependency>
 <groupId>org.mybatis</groupId>
 <artifactId>mybatis</artifactId>
 <version>3.4.5</version>
</dependency>
<!--mysql驱动坐标-->
<dependency> 
 <groupId>mysql</groupId> 
 <artifactId>mysql-connector-java</artifactId> 
 <version>5.1.6</version> 
 <scope>runtime</scope>
</dependency>
<!--单元测试坐标-->
<dependency> 
 <groupId>junit</groupId> 
 <artifactId>junit</artifactId> 
 <version>4.12</version> 
 <scope>test</scope>
</dependency>
<!--⽇志坐标-->
<dependency> 
 <groupId>log4j</groupId> 
 <artifactId>log4j</artifactId> 
 <version>1.2.12</version>
</dependency>
```

2. 創建user數據表
	![[Pasted image 20210927115848.png]] 
3. 編寫UserMapper映射⽂件
```java
public class User { 
 private int id; 
 private String username; 
 private String password;
 //省略get个set⽅法
}
```


4. 編寫UserMapper映射文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper 
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="userMapper"> 
 <select id="findAll" resultType="com.lagou.domain.User"> 
 	select * from User 
 </select>
</mapper>
```

5. 編寫MyBatis核心文件
```xml
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN“
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration> 
 <environments default="development"> 
	 <environment id="development"> 
	 <transactionManager type="JDBC"/> 
		 <dataSource type="POOLED"> 
		 <property name="driver" value="com.mysql.jdbc.Driver"/>
		 <property name="url" value="jdbc:mysql:///test"/> 
		 <property name="username" value="root"/>
		 <property name="password" value="root"/> 
	 </dataSource> 
	 </environment> 
 </environments> 
 
 <mappers>
 	<mapper resource="com/lagou/mapper/UserMapper.xml"/>
 </mappers>
</configuration>
```

6. 編寫測試代碼
```java
//加载核⼼配置⽂件
InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapConfig.xml");
//获得sqlSession⼯⼚对象
SqlSessionFactory sqlSessionFactory = new 
 SqlSessionFactoryBuilder().build(resourceAsStream);
//获得sqlSession对象
SqlSession sqlSession = sqlSessionFactory.openSession();
//执⾏sql语句
List<User> userList = sqlSession.selectList("userMapper.findAll");
//打印结果
System.out.println(userList);
//释放资源
sqlSession.close();
```

### 3.1.4 MyBatis的曾刪改查操作

MyBatis的插入數據操作

1. 編寫UserMapper映射文件
```xml
<mapper namespace="userMapper"> 
 <insert id="add" parameterType="com.lagou.domain.User"> 
 	insert into user values(#{id},#{username},#{password}) 
 </insert>
</mapper>
```

2. 編寫實體User的代碼
```java
InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapConfig.xml");
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
SqlSession sqlSession = sqlSessionFactory.openSession();
int insert = sqlSession.insert("userMapper.add", user);
System.out.println(insert);
//提交事务
sqlSession.commit();
sqlSession.close();
```

3. 插入操作注意問題
- 插入語句使用 insert 標籤
- 在映射⽂件中使⽤parameterType屬性指定要插⼊的數據類型
- Sql語句中使⽤#{實體屬性名}⽅式引⽤實體中的屬性值
- 插⼊操作使⽤的API是sqlSession.insert(“命名空間.id”,實體對象);
- 插⼊操作涉及數據庫數據變化，所以要使⽤sqlSession對象顯示的提交事務，即sqlSession.commit()


### 3.1.5 MyBatis的增刪改查操作

1. 編寫  UserMapper映射文件
```xml
<mapper namespace="userMapper">
 <update id="update" parameterType="com.lagou.domain.User">
 	update user set username=#{username},password=#{password} where id=#{id}
 </update>
</mapper>
```

2. 編寫修改實體User的代碼
```java
InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapConfig.xml");
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
SqlSession sqlSession = sqlSessionFactory.openSession();
int update = sqlSession.update("userMapper.update", user);
System.out.println(update);
sqlSession.commit();
sqlSession.close();
```

3. 修改操作注意問題
- 修改語句使用 update 標籤
- 修改操作使用API是sqlSession.update("命名空間.id", 實體對象)

### 3.1.6 MyBatis的刪除數據操作

1. 編寫  UserMapper映射文件
```xml
<mapper namespace="userMapper">
 <delete id="delete" parameterType="java.lang.Integer">
 delete from user where id=#{id}
 </delete>
</mapper>

```

2. 編寫刪除數據的代碼
```java
InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapConfig.xml");
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
SqlSession sqlSession = sqlSessionFactory.openSession();
int delete = sqlSession.delete("userMapper.delete",3);
System.out.println(delete);
sqlSession.commit();
sqlSession.close();
```

3. 刪除操作注意問題
- 刪除語句使用 delete 標籤
- Sql語句中使用#{任意字符串}方式引用傳遞的單個參數
- 修改操作使用API是sqlSession.delete("命名空間.id", Object)


### 3.1.6 MyBatis的映射文件概述

![[Pasted image 20210927151604.png]]


### 3.1.7 入門核心配置文件分析

**MyBatis核心配置文件層級關係**
![[Pasted image 20210927151709.png]]

**MyBatis常用配置解析**
1. environments標籤
數據庫環境的配置，支持多環境配置
![[Pasted image 20210927151858.png]]

其中，事務管理器（transactionManager）類型有兩種：
- JDBC：這個配置就是直接使⽤了JDBC 的提交和回滾設置，它依賴於從數據源得到的連接來管理事務作⽤域。
- MANAGED：這個配置⼏乎沒做什麼。它從來不提交或回滾⼀個連接，⽽是讓容器來管理事務的整個⽣命週期（⽐如 JEE 應⽤服務器的上下⽂）。默認情況下它會關閉連接，然⽽⼀些容器並不希望這樣，因此需要將closeConnection 屬性設置為 false 來阻⽌它默認的關閉⾏為。


其中，數據源（dataSource）類型有三種：
- UNPOOLED：這個數據源的實現只是每次被請求時打開和關閉連接。
- POOLED：這種數據源的實現利⽤“池”的概念將 JDBC 連接對象組織起來。
- JNDI：這個數據源的實現是為了能在如 EJB 或應⽤服務器這類容器中使⽤，容器可以集中或在外部配置數據源，然後放置⼀個 JNDI 上下⽂的引⽤。



2. mapper標籤
該標籤的作⽤是加載映射的，加載⽅式有如下⼏種：
```xml
•使⽤相对于类路径的资源引⽤，例如：
<mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
•使⽤完全限定资源定位符（URL），例如：
<mapper url="file:///var/mappers/AuthorMapper.xml"/>
•使⽤映射器接⼝实现类的完全限定类名，例如：
<mapper class="org.mybatis.builder.AuthorMapper"/>
•将包内的映射器接⼝实现全部注册为映射器，例如：
<package name="org.mybatis.builder"/>
```


### 3.1.7 Mybatis 相應API介紹

**SqlSession⼯⼚構建器SqlSessionFactoryBuilder**

常⽤API：SqlSessionFactory build(InputStream inputStream)

通過加載mybatis的核⼼⽂件的輸⼊流的形式構建⼀個SqlSessionFactory對象
```java
String resource = "org/mybatis/builder/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(inputStream);
```

其中，Resources ⼯具類，這個類在 org.apache.ibatis.io 包中。 Resources 類幫助你從類路徑下、⽂件系統或⼀個 web URL 中加載資源⽂件。

**SqlSession⼯⼚對象SqlSessionFactory**

SqlSessionFactory 有多個個⽅法創建SqlSession 實例。常⽤的有如下兩個：
![[Pasted image 20210927152615.png]]

**SqlSession會話對象**

SqlSession 實例在 MyBatis 中是⾮常強⼤的⼀個類。在這⾥你會看到所有執⾏語句、提交或回滾事務和獲取映射器實例的⽅法。

執⾏語句的⽅法主要有：

```java
<T> T selectOne(String statement, Object parameter)
<E> List<E> selectList(String statement, Object parameter)
int insert(String statement, Object parameter)
int update(String statement, Object parameter)
int delete(String statement, Object parameter)
```

操作事物的方法住要有：
```java
void commit() 
void rollback()
```


## 3.2 MyBatis 的 DAO層實現
### 3.2.1 傳統開發方式
編寫UserDao接口
```java
public interface UserDao {
 List<User> findAll() throws IOException;
}
```

編寫UserDaoImpl實現
```java
public class UserDaoImpl implements UserDao {
 public List<User> findAll() throws IOException {
	 InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapConfig.xml");
	 SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
	 SqlSession sqlSession = sqlSessionFactory.openSession();
	 List<User> userList = sqlSession.selectList("userMapper.findAll");
	 sqlSession.close();
	 return userList;
 }
}
```

測試傳統方式
```java
@Test
public void testTraditionDao() throws IOException {
 UserDao userDao = new UserDaoImpl();
 List<User> all = userDao.findAll();
 System.out.println(all);
}

```

### 3.2.2 代理開發方式
代理開發⽅式介紹

採⽤ Mybatis 的代理開發⽅式實現 DAO 層的開發，這種⽅式是我們後⾯進⼊企業的主流。

Mapper 接⼝開發⽅法只需要程序員編寫Mapper 接⼝（相當於Dao 接⼝），由Mybatis 框架根據接⼝定義創建接⼝的動態代理對象，代理對象的⽅法體同上邊Dao接⼝實現類⽅法。

Mapper 接⼝開發需要遵循以下規範：

1) Mapper.xml⽂件中的namespace與mapper接⼝的全限定名相同
2) Mapper接⼝⽅法名和Mapper.xml中定義的每個statement的id相同
3) Mapper接⼝⽅法的輸⼊參數類型和mapper.xml中定義的每個sql的parameterType的類型相同
4) Mapper接⼝⽅法的輸出參數類型和mapper.xml中定義的每個sql的resultType的類型相同


編寫UserMapper接⼝
![[Pasted image 20210927153358.png]]

測試代理方式
```java
@Test
public void testProxyDao() throws IOException {
 InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapConfig.xml");
 SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
 SqlSession sqlSession = sqlSessionFactory.openSession();
 //获得MyBatis框架⽣成的UserMapper接⼝的实现类
 UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
 User user = userMapper.findById(1);
 System.out.println(user);
 sqlSession.close();
}

```



























