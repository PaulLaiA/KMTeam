---
date: 2021-11-04
aliases: ["Mybatis註解開發"]
---

# Metadata

**Title** :: 自定義持久層框架

**Author** :: #Matthew 

**Classification** :: #Learn #Java #Basic

**Status** :: #🌱

**Type** :: #Note

**Previous** ::[[Study_Matthew/MyBatis/5_第五部分 Mybatis複雜映射開發]]

**ParentNode** :: [[Study_Matthew/MyBatis/MyBatis|MyBatis]]

---

# 第六部分： Mybatis 註解開發

## 6.1 MyBatis 的常用註解
這⼏年來註解開發越來越流⾏，Mybatis 也可以使⽤註解開發⽅式，這樣我們就可以減少編寫 Mapper 映射⽂件了。

我們先圍繞⼀些基本的 CRUD 來學習，再學習複雜映射多表操作。
@Insert：實現新增

@Update：實現更新

@Delete：實現刪除

@Select：實現查詢

@Result：實現結果集封裝

@Results：可以與@Result ⼀起使⽤，封裝多個結果集

@One：實現⼀對⼀結果集封裝

@Many：實現⼀對多結果集封裝


## 6.2 MyBatis 的增刪改查
我們完成簡單的 user 表的增刪改查的操作
```java
private UserMapper userMapper;
@Before
public void before() throws IOException {
 InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapConfig.xml");
 SqlSessionFactory sqlSessionFactory = new
 SqlSessionFactoryBuilder().build(resourceAsStream);
 SqlSession sqlSession = sqlSessionFactory.openSession(true);
 userMapper = sqlSession.getMapper(UserMapper.class);
}

@Test
public void testAdd() {
 User user = new User();
 user.setUsername("测试数据");
 user.setPassword("123");
 user.setBirthday(new Date());
 userMapper.add(user);
}
@Test
public void testUpdate() throws IOException {
 User user = new User();
 user.setId(16);
 user.setUsername("测试数据修改");
 user.setPassword("abc");
 user.setBirthday(new Date());
 userMapper.update(user);
}
@Test
public void testDelete() throws IOException {
 userMapper.delete(16);
}
@Test
public void testFindById() throws IOException {
 User user = userMapper.findById(1);
 System.out.println(user);
}
@Test
public void testFindAll() throws IOException {
 List<User> all = userMapper.findAll();
 for(User user : all){
 System.out.println(user);
 }
}
```
修改 MyBatis 的核⼼配置⽂件，我們使⽤了註解替代的映射⽂件，所以我們只需要加載使⽤了註解的 Mapper 接⼝即可
```xml
<mappers>
 <!--扫描使⽤注解的类-->
 <mapper class="com.lagou.mapper.UserMapper"></mapper>
</mappers>

```
或者指定掃描包含映射關係的接⼝所在的包也可以
```xml
<mappers>
 <!--扫描使⽤注解的类所在的包-->
 <package name="com.lagou.mapper"></package>
</mappers>
```


## 6.3 MyBatis 的註解實現複雜映射開發
實現複雜關係映射之前我們可以在映射⽂件中通過配置來實現，使⽤註解開發後，我們可以使⽤@Results 註解，@Result 註解，@One 註解，@Many 註解組合完成複雜關係的配置

![[Pasted image 20211002195830.png]]


## 6.4 一對一查詢
### 6.4.1 一對一查詢的模型
⽤戶表和訂單表的關係為，⼀個⽤戶有多個訂單，⼀個訂單只從屬於⼀個⽤戶

⼀對⼀查詢的需求：查詢⼀個訂單，與此同時查詢出該訂單所屬的⽤戶
![[Pasted image 20211002200034.png]]

### 6.4.2 ⼀對⼀查詢的語句 

對應的 sql 語句：
```sql
select * from orders; 
select * from user where id= 查询出订单的 uid
```

查詢的結果如下：
![[Pasted image 20211002200153.png]]


### 6.4.3 創建 Order 和 User 實體
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

### 6.4.4 創建 OrderMapper 接口
```java
public interface OrderMapper {
 List<Order> findAll();
}
```

### 6.4.5 使用註解配置 Mapper
```java
public interface OrderMapper {
 @Select("select * from orders")
 @Results({
	 @Result(id=true,property = "id",column = "id"),
	 @Result(property = "ordertime",column = "ordertime"),
	 @Result(property = "total",column = "total"),
	 @Result(property = "user",column = "uid",
						 javaType = User.class,
						 one = @One(select = "com.lagou.mapper.UserMapper.findById"))
 })
 List<Order> findAll();
}
```

```java
public interface UserMapper {
 @Select("select * from user where id=#{id}")
 User findById(int id);
}
```

### 6.4.6 測試結果
```java
@Test
public void testSelectOrderAndUser() {
 List<Order> all = orderMapper.findAll();
 for(Order order : all){
 	System.out.println(order);
 }
}
```
![[Pasted image 20211002200734.png]]


## 6.5 一對多查詢
### 6.5.1 一對一查詢的模型
⽤戶表和訂單表的關係為，⼀個⽤戶有多個訂單，⼀個訂單只從屬於⼀個⽤戶

⼀對多查詢的需求：查詢⼀個⽤戶，與此同時查詢出該⽤戶具有的訂單
![[Pasted image 20211002200833.png]]

### 6.5.2 ⼀對多查詢的語句 

對應的 sql 語句：
```sql
select * from user; 
select * from orders where uid= 查询出⽤户的 id;
```

查詢的結果如下：
![[Pasted image 20211002200914.png]]


### 6.5.3 修改 User 實體
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

### 6.5.4 創建 UserMapper 接口
```java
public interface UserMapper {
 List findAllUserAndOrder()
}
```

### 6.5.5 使用註解配置 Mapper
```java
public interface UserMapper {
 @Select("select * from user")
 @Results({
	 @Result(id = true,property = "id",column = "id"),
	 @Result(property = "username",column = "username"),
	 @Result(property = "password",column = "password"),
	 @Result(property = "birthday",column = "birthday"),
	 @Result(property = "orderList",column = "id",
					 javaType = List.class,
					 many = @Many(select = "com.lagou.mapper.OrderMapper.findByUid"))
 })
 List<User> findAllUserAndOrder();
}
public interface OrderMapper {
 @Select("select * from orders where uid=#{uid}")
 List<Order> findByUid(int uid);
}
```

### 6.5.6 測試結果
```java
List<User> all = userMapper.findAllUserAndOrder();
for(User user : all){
 System.out.println(user.getUsername());
 List<Order> orderList = user.getOrderList();
 for(Order order : orderList){
 System.out.println(order);
 }
 System.out.println("-----------------------------");
}

```
![[Pasted image 20211002201143.png]]




## 6.6 多對多查詢
### 6.6.1 多對多查詢的模型
⽤戶表和⻆⾊表的關係為，⼀個⽤戶有多個⻆⾊，⼀個⻆⾊被多個⽤戶使⽤

多對多查詢的需求：查詢⽤戶同時查詢出該⽤戶的所有⻆⾊
![[Pasted image 20211002201300.png]]

### 6.6.2 ⼀對多查詢的語句 

對應的 sql 語句：
```sql
select * from user; 
select * from role r,user_role ur where r.id=ur.role_id and ur.user_id= ⽤户的 id
```

查詢的結果如下：
![[Pasted image 20211002201327.png]]


### 6.6.3 創建 Role 實體，修改 User 實體
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

### 6.6.4 添加 UserMapper 接口方法
```java
List findAllUserAndRole();
```

### 6.6.5 使用註解配置 Mapper
```java
public interface UserMapper {
 @Select("select * from user")
 @Results({
	 @Result(id = true,property = "id",column = "id"),
	 @Result(property = "username",column = "username"),
	 @Result(property = "password",column = "password"),
	 @Result(property = "birthday",column = "birthday"),
	 @Result(property = "roleList",column = "id",
					 javaType = List.class,
					 many = @Many(select = "com.lagou.mapper.RoleMapper.findByUid"))
})
List<User> findAllUserAndRole();}
public interface RoleMapper {
 @Select("select * from role r,user_role ur where r.id=ur.role_id and ur.user_id=#
{uid}")
 List<Role> findByUid(int uid);
}
```

### 6.6.6 測試結果
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
![[Pasted image 20211002201621.png]]


