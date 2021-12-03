---
date: 2021-11-04
aliases: ["Mybatis插件"]
---

# Metadata

**Title** :: 自定義持久層框架

**Author** :: #Matthew 

**Classification** :: #Learn #Java #Basic

**Status** :: #🌱

**Type** :: #Note

**Previous** ::[[Study_Matthew/MyBatis/7_第七部分 Mybatis緩存]]

**ParentNode** :: [[Study_Matthew/MyBatis/MyBatis|MyBatis]]

---


# 第八部分： Mybatis 插件

## 8.1 插件簡介

⼀般情況下，開源框架都會提供插件或其他形式的拓展點，供開發者⾃⾏拓展。這樣的好處是顯⽽易⻅ 的，⼀是增加了框架的靈活性。 ⼆是開發者可以結合實際需求，對框架進⾏拓展，使其能夠更好的⼯ 作。

以 MyBatis 為例，我們可基於 MyBati s 插件機制實現分⻚、分錶，監控等功能。由於插件和業務 ⽆關，業務也⽆法感知插件的存在。因此可以⽆感植⼊插件，在⽆形中增強功能


## 8.2 Mybatis 插件介绍

Mybati s 作為⼀個應⽤⼴泛的優秀的 ORM 開源框架，這個框架具有強⼤的靈活性，在四⼤組件

(Executor、StatementHandler、ParameterHandler、ResultSetHandler)處提供了簡單易⽤的插件擴展機制。

Mybatis 對持久層的操作就是藉助於四⼤核⼼對象。 MyBatis ⽀持⽤插件對四⼤核⼼對象進⾏攔截，對 mybatis 來說插件就是攔截器，⽤來增強核⼼對象的功能，增強功能本質上是藉助於底層的 動態代理實現的，

換句話說，MyBatis 中的四⼤對像都是代理對象

![[Pasted image 20211003142849.png]]

**MyBatis 所允許攔截的⽅法如下：**
- 執⾏器 Executor (update、query、commit、rollback 等⽅法)；
- SQL 語法構建器 StatementHandler (prepare、parameterize、batch、updates query 等⽅ 法)；
- 參數處理器 ParameterHandler (getParameterObject、setParameters ⽅法)；
- 結果集處理器 ResultSetHandler (handleResultSets、handleOutputParameters 等⽅法)；

## 8.3 Mybatis 插件原理
在四⼤對象創建的時候
1. 每個創建出來的對像不是直接返回的，⽽是 interceptorChain.pluginAll(parameterHandler);
2. 獲取到所有的 Interceptor (攔截器)(插件需要實現的接⼝)；調⽤ interceptor.plugin(target);返回 target 包裝後的對象
3. 插件機制，我們可以使⽤插件為⽬標對象創建⼀個代理對象；AOP (⾯向切⾯)我們的插件可 以為四⼤對象創建出代理對象，代理對象就可以攔截到四⼤對象的每⼀個執⾏；

**攔截**

插件具體是如何攔截並附加額外的功能的呢？以 ParameterHandler 來說
```java
public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object object, BoundSql sql, InterceptorChain interceptorChain){
 ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement,object,sql);
 parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
 return parameterHandler;
}
public Object pluginAll(Object target) {
 for (Interceptor interceptor : interceptors) {
 target = interceptor.plugin(target);
 }
 return target;
}
```

interceptorChain 保存了所有的攔截器(interceptors)，是 mybatis 初始化的時候創建的。調⽤攔截器鏈 中的攔截器依次的對⽬標進⾏攔截或增強。

interceptor.plugin(target)中的 target 就可以理解為 mybatis 中的四⼤對象。返回的 target 是被重重代理後的對象

如果我們想要攔截 Executor 的 query ⽅法，那麼可以這樣定義插件：
```java
@Intercepts({
 @Signature(
	 type = Executor.class,
	 method = "query",
	 args={MappedStatement.class,Object.class,RowBounds.class,ResultHandler.class}
	 )
}) 
public class ExamplePlugin implements Interceptor {
 //省略邏輯
}
```

除此之外，我們還需將插件配置到 sqlMapConfig.xm l 中。
```xml
<plugins>
 <plugin interceptor="com.lagou.plugin.ExamplePlugin">
 </plugin>
</plugins>
```

這樣 MyBatis 在啟動時可以加載插件，並保存插件實例到相關對象(InterceptorChain，攔截器鏈) 中。待準備⼯作做完後，MyBatis 處於就緒狀態。

我們在執⾏ SQL 時，需要先通過 DefaultSqlSessionFactory 創建 SqlSession。

Executor 實例會在創建 SqlSession 的過程中被創建， Executor 實例創建完畢後，MyBatis 會通過 JDK 動態代理為實例⽣成代理類。這樣，插件邏輯即可在 Executor 相關⽅法被調⽤前執⾏。

以上就是 MyBatis 插件機制的基本原理


## 8.4 自定義插件

### 8.4.1 插件接口

Mybatis 插件接⼝-Interceptor

• Intercept ⽅法，插件的核⼼⽅法

• plugin ⽅法，⽣成 target 的代理對象

• setProperties ⽅法，傳遞插件所需參數


### 8.4.2 自定義插件

設計實現⼀個⾃定義插件

```java
Intercepts ({//注意看这个⼤花括号，也就这说这⾥可以定义多个@Signature 对多个地⽅拦截，都⽤这个拦截器
@Signature (type = StatementHandler .class , //这是指拦截哪个接⼝
 method = "prepare"，//这个接⼝内的哪个⽅法名，不要拼错了
 args = { Connection.class, Integer .class}),//// 这是拦截的⽅法的⼊参，按顺序写到这，不要多也不要少，如果⽅法重载，可是要通过⽅法名和⼊参来确定唯⼀的
})
public class MyPlugin implements Interceptor {
 private final Logger logger = LoggerFactory.getLogger(this.getClass());
 // //这⾥是每次执⾏操作的时候，都会进⾏这个拦截器的⽅法内
 
 @Override
 public Object intercept(Invocation invocation) throws Throwable {
 	//增强逻辑
 	System.out.println("对⽅法进⾏了增强....")；
 	return invocation.proceed(); //执⾏原⽅法
}
 
 /**
 * //主要是为了把这个拦截器⽣成⼀个代理放到拦截器链中
 * ^Description 包装⽬标对象 为⽬标对象创建代理对象
 * @Param target 为要拦截的对象
 * @Return 代理对象
 */
 @Override
 public Object plugin(Object target) {
	 System.out.println("将要包装的⽬标对象："+target);
	 return Plugin.wrap(target,this);
 }
 
 /**获取配置⽂件的属性**/
 //插件初始化的时候调⽤，也只调⽤⼀次，插件配置的属性从这⾥设置进来
 @Override
 public void setProperties(Properties properties) {
 	System.out.println("插件配置的初始化参数："+properties );
 }
}

```

sqlMapConfig.xml

```xml
<plugins>
 <plugin interceptor="com.lagou.plugin.MySqlPagingPlugin">
 <!--配置参数-->
 <property name="name" value="Bob"/>
 </plugin>
</plugins>
```

mapper 接口
```java
public interface UserMapper {
 List<User> selectUser();
}
```

mapper.xml
```xml
<mapper namespace="com.lagou.mapper.UserMapper">
 <select id="selectUser" resultType="com.lagou.pojo.User">
 SELECT
 id,username
 FROM
 user
 </select>
</mapper>
```

測試類
```java
public class PluginTest {
	@Test
	public void test() throws IOException {
	 InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
	 SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
	 SqlSession sqlSession = sqlSessionFactory.openSession();
	 UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
	 List<User> byPaging = userMapper.selectUser();
	 for (User user : byPaging) {
		System.out.println(user);
	 }
 }
}
```

## 8.5 源碼分析

執⾏插件邏輯

Plugin 實現了 InvocationHandler 接⼝，因此它的 invoke ⽅法會攔截所有的⽅法調⽤。 invoke ⽅法會 對所攔截的⽅法進⾏檢測，以決定是否執⾏插件邏輯。該⽅法的邏輯如下：

```java
// -Plugin
 public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	 try {
		 /*
		 *获取被拦截⽅法列表，⽐如：
		 * signatureMap.get(Executor.class), 可能返回 [query, update, commit]
		 */
		 Set<Method> methods = signatureMap.get(method.getDeclaringClass());
		 //检测⽅法列表是否包含被拦截的⽅法
		 if (methods != null && methods.contains(method)) {
		 //执⾏插件逻辑
		 return interceptor.intercept(new Invocation(target, method, args));
		 //执⾏被拦截的⽅法
		 return method.invoke(target, args);
	 } catch(Exception e){
	 }
 }
```

invoke ⽅法的代碼⽐較少，邏輯不難理解。 ⾸先,invoke ⽅法會檢測被攔截⽅法是否配置在插件的 @Signature 註解中，若是，則執⾏插件邏輯，否則執⾏被攔截⽅法。

插件邏輯封裝在 intercept 中，該⽅法的參數類型為 Invocationo Invocation 主要⽤於存儲⽬標類，⽅法以及⽅法參數列表。下⾯簡單看 ⼀下該類的定義
```java
public class Invocation {
private final Object target;
private final Method method;
private final Object[] args;
public Invocation(Object targetf Method method, Object[] args) {
this.target = target;
this.method = method;
//省略部分代码
public Object proceed() throws InvocationTargetException, IllegalAccessException { //调
⽤被拦截的⽅法
>> —
```
關於插件的執⾏邏輯就分析結束


## 8.6 pageHelper 分頁插件
MyBati s 可以使⽤第三⽅的插件來對功能進⾏擴展，分⻚助⼿ PageHelper 是將分⻚的複雜操作進⾏封裝，使⽤簡單的⽅式即可獲得分⻚的相關數據

開發步驟：

① 導⼊通⽤ PageHelper 的坐標

② 在 mybatis 核⼼配置⽂件中配置 PageHelper 插件

③ 測試分⻚數據獲取



**①導⼊通⽤ PageHelper 坐標**
```xml
 <dependency>
	 <groupId>com.github.pagehelper</groupId>
	 <artifactId>pagehelper</artifactId>
	 <version>3.7.5</version>
 </dependency>
 <dependency>
	 <groupId>com.github.jsqlparser</groupId>
	 <artifactId>jsqlparser</artifactId>
	 <version>0.9.1</version>
 </dependency>
```

**② 在 mybatis 核⼼配置⽂件中配置 PageHelper 插件**
```xml
<!--注意：分⻚助⼿的插件 配置在通⽤馆 mapper 之前*-->*
 <plugin interceptor="com.github.pagehelper.PageHelper">
	 <!—指定⽅⾔ —>
	 <property name="dialect" value="mysql"/>
 </plugin>
```

**③ 測試分⻚代碼實現**
```java
//其他分⻚的数据
PageInfo<User> pageInfo = new PageInfo<User>(select);
System.out.println("总条数："+pageInfo.getTotal());
System.out.println("总⻚数："+pageInfo. getPages ());
System.out.println("当前⻚："+pageInfo. getPageNum());
System.out.println("每⻚显万⻓度："+pageInfo.getPageSize());
System.out.println("是否第⼀⻚："+pageInfo.isIsFirstPage());
System.out.println("是否最后⼀⻚："+pageInfo.isIsLastPage());
```


## 8.7 通用 mapper
**什麼是通⽤ Mapper**

通⽤ Mapper 就是為了解決單表增刪改查，基於 Mybatis 的插件機制。開發⼈員不需要編寫 SQL,不需要 在 DAO 中增加⽅法，只要寫好實體類，就能⽀持相應的增刪改查⽅法

**如何使⽤**

1. ⾸先在 maven 項⽬，在 pom.xml 中引⼊ mapper 的依賴
```xml
<dependency>
 <groupId>tk.mybatis</groupId>
 <artifactId>mapper</artifactId>
 <version>3.1.2</version>
</dependency>
```

2. Mybatis 配置⽂件中完成配置
```xml
<plugins>
 <!--分⻚插件：如果有分⻚插件，要排在通⽤ mapper 之前-->
 <plugin interceptor="com.github.pagehelper.PageHelper">
 	<property name="dialect" value="mysql"/>
 </plugin>
 <plugin interceptor="tk.mybatis.mapper.mapperhelper.MapperInterceptor">
 <!-- 通⽤ Mapper 接⼝，多个通⽤接⼝⽤逗号隔开 -->
 	<property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
 </plugin>
</plugins>
```

3. 實體類設置主鍵
```java
@Table(name = "t_user")
public class User {
 @Id
 @GeneratedValue(strategy = GenerationType.IDENTITY)
 private Integer id;
 private String username;
}
```

4. 定義通用 mapper
```java
import com.lagou.domain.User;
import tk.mybatis.mapper.common.Mapper;
public interface UserMapper extends Mapper<User> {
}
```

5. 測試
```java
public class UserTest {
 @Test
 public void test1() throws IOException {
	 Inputstream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
	 SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resourceAsStream);
	 SqlSession sqlSession = build.openSession();
	 UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
	 User user = new User();
	 user.setId(4);
	 //(1)mapper 基础接⼝
	 //select 接⼝
	 User user1 = userMapper.selectOne(user); //根据实体中的属性进⾏查询，只能有 —个返回值
	 List<User> users = userMapper.select(null); //查询全部结果
	 userMapper.selectByPrimaryKey(1); //根据主键字段进⾏查询，⽅法参数必须包含完 整的主键属性，查询条件使⽤等号
	 userMapper.selectCount(user); //根据实体中的属性查询总数，查询条件使⽤等号
	 // insert 接⼝
	 int insert = userMapper.insert(user); //保存⼀个实体，null 值也会保存，不会使 ⽤数据库默认值
	 int i = userMapper.insertSelective(user); //保存实体，null 的属性不会保存， 会使⽤数据库默认值
	 // update 接⼝
	 int i1 = userMapper.updateByPrimaryKey(user);//根据主键更新实体全部字段， null 值会被更新
	 // delete 接⼝
	 int delete = userMapper.delete(user); //根据实体属性作为条件进⾏删除，查询条件 使⽤等号
	 userMapper.deleteByPrimaryKey(1); //根据主键字段进⾏删除，⽅法参数必须包含完 整的主键属性
	 //(2)example ⽅法
	 Example example = new Example(User.class);
	 example.createCriteria().andEqualTo("id", 1);
	 example.createCriteria().andLike("val", "1");
	 //⾃定义查询
	 List<User> users1 = userMapper.selectByExample(example);
 }
}
```















