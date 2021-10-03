---
界: Framework
門: MyBatis
綱: 
tags: ["#MyBatis","#BQ"]
aliases:
  - 
date: {{DATE:YYYY-MM-DD}}
---

# 第四部分：Mybatis配置⽂件深⼊

## 4.1 核⼼配置⽂件SqlMapConfig.xml

### 4.1.1 MyBatis核心配置文件層級關係
![[Pasted image 20210927153736.png]]


## 4.1 MyBatis常⽤配置解析
1) environments標籤
數據庫環境的配置，⽀持多環境配置
![[Pasted image 20210927153957.png]]
其中，事務管理器（transactionManager）類型有兩種：

•JDBC：這個配置就是直接使⽤了JDBC 的提交和回滾設置，它依賴於從數據源得到的連接來管理事務作⽤域。
•MANAGED：這個配置⼏乎沒做什麼。它從來不提交或回滾⼀個連接，⽽是讓容器來管理事務的整個⽣命週期（⽐如 JEE 應⽤服務器的上下⽂）。默認情況下它會關閉連接，然⽽⼀些容器並不希望這樣，因此需要將closeConnection 屬性設置為 false 來阻⽌它默認的關閉⾏為。

其中，數據源（dataSource）類型有三種：
•UNPOOLED：這個數據源的實現只是每次被請求時打開和關閉連接。
•POOLED：這種數據源的實現利⽤“池”的概念將 JDBC 連接對象組織起來。
•JNDI：這個數據源的實現是為了能在如 EJB 或應⽤服務器這類容器中使⽤，容器可以集中或在外部配置數據源，然後放置⼀個 JNDI 上下⽂的引⽤。

2) mapper標籤
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

3) Properties標籤
實際開發中，習慣將數據源的配置信息單獨抽取成⼀個properties⽂件，該標籤可以加載額外配置的properties⽂件
 ![[Pasted image 20210927154132.png]]
 
4) typeAliases標籤
類型別名是為Java 類型設置⼀個短的名字。原來的類型名稱配置如下
 ![[Pasted image 20210927154152.png]]
 
配置typeAliases，為com.lagou.domain.User定義別名為user
![[Pasted image 20210927154210.png]]

上⾯我們是⾃定義的別名，mybatis框架已經為我們設置好的⼀些常⽤的類型的別名
![[Pasted image 20210927154230.png]]


## 4.2 映射配置文件 mapper.xml
**動態sql語句**

**動態sql語句概述**

Mybatis 的映射⽂件中，前⾯我們的 SQL 都是⽐較簡單的，有些時候業務邏輯複雜時，我們的 SQL是動態變化的，此時在前⾯的學習中我們的 SQL 就不能滿⾜要求了。

參考的官⽅⽂檔，描述如下：
![[Pasted image 20210927154352.png]]

**動態 SQL 之** 

我們根據實體類的不同取值，使⽤不同的 SQL語句來進⾏查詢。 ⽐如在 id如果不為空時可以根據id查詢，如果username 不同空時還要加⼊⽤戶名作為條件。這種情況在我們的多條件組合查詢中經常會碰到。
```xml
<select id="findByCondition" parameterType="user" resultType="user">
 select * from User
 <where>
 <if test="id!=0">
 and id=#{id}
 </if>
 <if test="username!=null">
 and username=#{username}
 </if>
 </where>
</select>
```

當查詢條件id和username都存在時，控制台打印的sql語句如下：
```java
… … …
 //获得MyBatis框架⽣成的UserMapper接⼝的实现类
 UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
 User condition = new User();
 condition.setId(1);
 condition.setUsername("lucy");
 User user = userMapper.findByCondition(condition);
 … … …
```
![[Pasted image 20210927154602.png]]

當查詢條件只有id存在時，控制台打印的sql語句如下：
```java
… … …
//获得MyBatis框架⽣成的UserMapper接⼝的实现类
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
User condition = new User();
condition.setId(1);
User user = userMapper.findByCondition(condition);
… … …

```
![[Pasted image 20210927154701.png]]

**動態 SQL 之**

循環執⾏sql的拼接操作，例如：SELECT * FROM USER WHERE id IN (1,2,5)。
```xml
<select id="findByIds" parameterType="list" resultType="user">
 select * from User
 <where>
 <foreach collection="list" open="id in(" close=")" item="id" separator=",">
 #{id}
 </foreach>
 </where>
</select>
```

測試代碼⽚段如下：
```java
… … …
//获得MyBatis框架⽣成的UserMapper接⼝的实现类
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
int[] ids = new int[]{2,5};
List<User> userList = userMapper.findByIds(ids);
System.out.println(userList);
… … …

```
![[Pasted image 20210927154838.png]]

foreach標籤的屬性含義如下：

標籤⽤於遍歷集合，它的屬性：
- collection：代表要遍歷的集合元素，注意編寫時不要寫#{}
- open：代表語句的開始部分
- close：代表結束部分
- item：代表遍歷集合的每個元素，⽣成的變量名
- sperator：代表分隔符

SQL⽚段抽取

Sql 中可將重複的 sql 提取出來，使⽤時⽤ include 引⽤即可，最終達到 sql 重⽤的⽬的
```xml
<!--抽取sql⽚段简化编写-->
<sql id="selectUser" select * from User</sql>
<select id="findById" parameterType="int" resultType="user">
 <include refid="selectUser"></include> where id=#{id}
</select>
<select id="findByIds" parameterType="list" resultType="user">
 <include refid="selectUser"></include>
 <where>
 <foreach collection="array" open="id in(" close=")" item="id" separator=",">
 #{id}
 </foreach>
 </where>
</select>
```

















