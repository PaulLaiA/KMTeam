---
界: Framework
門: MyBatis
綱: 
tags: ["#MyBatis","#BQ"]
aliases:
  - 
date: {{DATE:YYYY-MM-DD}}
---

# 第五部分：Mybatis 複雜映射開發

## 5.1 一對一查詢

### 5.1.1 一對一查詢的模型

⽤戶表和訂單表的關係為，⼀個⽤戶有多個訂單，⼀個訂單只從屬於⼀個⽤戶

⼀對⼀查詢的需求：查詢⼀個訂單，與此同時查詢出該訂單所屬的⽤戶
![[Pasted image 20210927155605.png]]

### 5.1.2 ⼀對⼀查詢的語句 

對應的 sql 語句：select * from orders o,user u where o.uid=u.id; 

查詢的結果如下：
![[Pasted image 20210927155657.png]]

### 5.1.3 創建 Order 和 User 實體
```java
public class Order {
 private int id;
 private Date ordertime;
 private double total;
 //代表当前订单从属于哪⼀个客户
 private User user;
}
public class User {
 
 private int id;
 private String username;
 private String password;
 private Date birthday;
}
```

### 5.1.4 創建 OrderMapper 接口
```java
public interface OrderMapper {
 List<Order> findAll();
}
```

### 5.1.5 配置 OrderMapper.xml
```xml
<mapper namespace="com.lagou.mapper.OrderMapper">
 <resultMap id="orderMap" type="com.lagou.domain.Order">
 <result column="uid" property="user.id"></result>
 <result column="username" property="user.username"></result>
 <result column="password" property="user.password"></result>
 <result column="birthday" property="user.birthday"></result>
 </resultMap>
 <select id="findAll" resultMap="orderMap">
 select * from orders o,user u where o.uid=u.id
 </select>
</mapper>
```

其中還可以配置如下
```xml
<resultMap id="orderMap" type="com.lagou.domain.Order">
 <result property="id" column="id"></result>
 <result property="ordertime" column="ordertime"></result>
 <result property="total" column="total"></result>
 <association property="user" javaType="com.lagou.domain.User">
 <result column="uid" property="id"></result>
 <result column="username" property="username"></result>
 <result column="password" property="password"></result>
 <result column="birthday" property="birthday"></result>
 </association>
</resultMap>

```

### 5.1.6 測試結果
```java
OrderMapper mapper = sqlSession.getMapper(OrderMapper.class);
List<Order> all = mapper.findAll();
for(Order order : all){
 System.out.println(order);
}
```

## 5.2 一對多查詢
### 5.2.1 一對多查詢的模型
⽤戶表和訂單表的關係為，⼀個⽤戶有多個訂單，⼀個訂單只從屬於⼀個⽤戶

⼀對多查詢的需求：查詢⼀個⽤戶，與此同時查詢出該⽤戶具有的訂單
![[Pasted image 20210927160119.png]]

### 5.2.2 一對多查詢的語句
對應的 sql 語句：select*,o.id oid from user u left join orders o on u.id=o.uid;

查詢的結果如下：
![[Pasted image 20210927160218.png]]

### 5.2.3 修改 User 實體
```java
public class Order {
 private int id;
 private Date ordertime;
 private double total;
 //代表当前订单从属于哪⼀个客户
 private User user;
}
public class User {
 
 private int id;
 private String username;
 private String password;
 private Date birthday;
 //代表当前⽤户具备哪些订单
 private List<Order> orderList;
}
```

### 5.2.4 創建 UserMapper 接⼝
```java
public interface UserMapper {
 List<User> findAll();
}

```

### 5.2.5 配置 UserMapper.xml
```xml
<mapper namespace="com.lagou.mapper.UserMapper">
 <resultMap id="userMap" type="com.lagou.domain.User">
 <result column="id" property="id"></result>
 <result column="username" property="username"></result>
 <result column="password" property="password"></result>
 <result column="birthday" property="birthday"></result>
 <collection property="orderList" ofType="com.lagou.domain.Order">
 <result column="oid" property="id"></result>
 <result column="ordertime" property="ordertime"></result>
 <result column="total" property="total"></result>
 </collection>
 </resultMap>
 <select id="findAll" resultMap="userMap">
 select *,o.id oid from user u left join orders o on u.id=o.uid
 </select>
</mapper>
```

### 5.2.6 測試結果
```java
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
List<User> all = mapper.findAll();
for(User user : all){
 System.out.println(user.getUsername());
 List<Order> orderList = user.getOrderList();
 for(Order order : orderList){
 System.out.println(order);
 }
 System.out.println("----------------------------------");
}
```
![[Pasted image 20210927160503.png]]


## 5.3 多對多查詢
### 5.3.1 多對多查詢的模型
⽤戶表和⻆⾊表的關係為，⼀個⽤戶有多個⻆⾊，⼀個⻆⾊被多個⽤戶使⽤

多對多查詢的需求：查詢⽤戶同時查詢出該⽤戶的所有⻆⾊
![[Pasted image 20210927160556.png]]

### 5.3.2 多對多查詢的語句
對應的 sql 語句：select u.,r.,r.id rid from user u left join user_role ur on u.id=ur.user_id

 inner join role r on ur.role_id=r.id;
 
查詢的結果如下：
![[Pasted image 20210927160639.png]]

### 5.3.3 創建 Role 實體，修改 User 實體
```java
public class User {
 private int id;
 private String username;
 private String password;
 private Date birthday;
 //代表当前⽤户具备哪些订单
 private List<Order> orderList;
 //代表当前⽤户具备哪些⻆⾊
 private List<Role> roleList;
}
public class Role {
 private int id;
 private String rolename;
}
```

### 5.3.4 添加 UserMapper 接⼝⽅法
```java
List<User> findAllUserAndRole();
```


### 5.3.5 配置 UserMapper.xml
```xml
<resultMap id="userRoleMap" type="com.lagou.domain.User">
 <result column="id" property="id"></result>
 <result column="username" property="username"></result>
 <result column="password" property="password"></result>
 <result column="birthday" property="birthday"></result>
 <collection property="roleList" ofType="com.lagou.domain.Role">
 <result column="rid" property="id"></result>
 <result column="rolename" property="rolename"></result>
 </collection>
</resultMap>
<select id="findAllUserAndRole" resultMap="userRoleMap">
 select u.*,r.*,r.id rid from user u left join user_role ur on u.id=ur.user_id
 inner join role r on ur.role_id=r.id
</select>
```

### 5.3.6 測試結果
```java
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
List<User> all = mapper.findAllUserAndRole();
for(User user : all){
 System.out.println(user.getUsername());
 List<Role> roleList = user.getRoleList();
 for(Role role : roleList){
 System.out.println(role);
 }
 System.out.println("----------------------------------");
}
```
![[Pasted image 20210927161037.png]]


## 5.4 知識小結
MyBatis 多表配置⽅式：

**⼀對⼀配置：使⽤做配置**

**⼀對多配置：使⽤ + 做配置**

**多對多配置：使⽤ + 做配置**


















