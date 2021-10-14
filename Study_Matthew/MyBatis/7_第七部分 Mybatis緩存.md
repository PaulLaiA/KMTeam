---
界: Framework
門: MyBatis
綱: 
tags: ["#MyBatis","#BQ"]
aliases:
  - 
date: {{DATE:YYYY-MM-DD}}
---


# 第七部分： Mybatis緩存

## 7.1 一級緩存

①、在⼀個sqlSession中，對User表根據id進⾏兩次查詢，查看他們發出sql語句的情況

```java
@Test
public void test1(){
//根据 sqlSessionFactory 产⽣ session
 SqlSession sqlSession = sessionFactory.openSession();
 UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
 //第⼀次查询，发出sql语句，并将查询出来的结果放进缓存中
 User u1 = userMapper.selectUserByUserId(1);
 System.out.println(u1);
 //第⼆次查询，由于是同⼀个sqlSession,会在缓存中查询结果
 //如果有，则直接从缓存中取出来，不和数据库进⾏交互
 User u2 = userMapper.selectUserByUserId(1);
 System.out.println(u2);
 sqlSession.close();
}

```
查看控制台打印情況：
![[Pasted image 20211002202142.png]]

② 、同樣是對user表進⾏兩次查詢，只不過兩次查詢之間進⾏了⼀次update操作。

```java
@Test
public void test2(){
 //根据 sqlSessionFactory 产⽣ session
 SqlSession sqlSession = sessionFactory.openSession();
 UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
 //第⼀次查询，发出sql语句，并将查询的结果放⼊缓存中
 User u1 = userMapper.selectUserByUserId( 1 );
 System.out.println(u1);
 //第⼆步进⾏了⼀次更新操作，sqlSession.commit()
 u1.setSex("⼥");
 userMapper.updateUserByUserId(u1);
 sqlSession.commit();
 //第⼆次查询，由于是同⼀个sqlSession.commit(),会清空缓存信息
 //则此次查询也会发出sql语句
 User u2 = userMapper.selectUserByUserId(1);
 System.out.println(u2);
 sqlSession.close();
}

```
查看控制台打印情況：
![[Pasted image 20211002202247.png]]

③、總結

1、第⼀次發起查詢⽤戶id為1的⽤戶信息，先去找緩存中是否有id為1的⽤戶信息，如果沒有，從 數據庫查詢⽤戶信息。得到⽤戶信息，將⽤戶信息存儲到⼀級緩存中。

2、 如果中間sqlSession去執⾏commit操作（執⾏插⼊、更新、刪除），則會清空SqlSession中的 ⼀級緩存，這樣做的⽬的為了讓緩存中存儲的是最新的信息，避免臟讀。

3、 第⼆次發起查詢⽤戶id為1的⽤戶信息，先去找緩存中是否有id為1的⽤戶信息，緩存中有，直 接從緩存中獲取⽤戶信息

![[Pasted image 20211002202444.png]]

**⼀級緩存原理探究與源碼分析**

⼀級緩存到底是什麼？ ⼀級緩存什麼時候被創建、⼀級緩存的⼯作流程是怎樣的？相信你現在應該會有 這⼏個疑問，那麼我們本節就來研究⼀下⼀級緩存的本質

⼤家可以這樣想，上⾯我們⼀直提到⼀級緩存，那麼提到⼀級緩存就繞不開SqlSession,所以索性我們 就直接從SqlSession，看看有沒有創建緩存或者與緩存有關的屬性或者⽅法
![[Pasted image 20211002202613.png]]

調研了⼀圈，發現上述所有⽅法中，好像只有clearCache()和緩存沾點關係，那麼就直接從這個⽅ 法⼊⼿吧，分析源碼時，我們要看它(此類)是誰，它的⽗類和⼦類分別⼜是誰，對如上關係了解了，你才 會對這個類有更深的認識，分析了⼀圈，你可能會得到如下這個流程圖
![[Pasted image 20211002202653.png]]

再深⼊分析，流程⾛到Perpetualcache中的clear()⽅法之後，會調⽤其cache.clear()⽅法，那 麼這個cache是什麼東⻄呢？

點進去發現，cache其實就是private Map cache = newHashMap()；也就是⼀個Map，所以說cache.clear()其實就是map.clear()，也就是說，緩存其實就是 本地存放的⼀個map對象，每⼀個SqISession都會存放⼀個map對象的引⽤，那麼這個cache是何 時創建的呢？

你覺得最有可能創建緩存的地⽅是哪⾥呢？我覺得是Executor，為什麼這麼認為？因為Executor是 執⾏器，⽤來執⾏SQL請求，⽽且清除緩存的⽅法也在Executor中執⾏，所以很可能緩存的創建也很 有可能在Executor中，看了⼀圈發現Executor中有⼀個createCacheKey⽅法，這個⽅法很像是創 建緩存的⽅法啊，跟進去看看，你發現createCacheKey⽅法是由BaseExecutor執⾏的，代碼如下
```java
CacheKey cacheKey = new CacheKey();
//MappedStatement 的 id
// id就是Sql语句的所在位置包名+类名+ SQL名称
cacheKey.update(ms.getId());
// offset 就是 0
cacheKey.update(rowBounds.getOffset());
// limit 就是 Integer.MAXVALUE
cacheKey.update(rowBounds.getLimit());
//具体的SQL语句
cacheKey.update(boundSql.getSql());
//后⾯是update 了 sql中带的参数
cacheKey.update(value);
...
if (configuration.getEnvironment() != null) {
// issue #176
cacheKey.update(configuration.getEnvironment().getId());
}

```

創建緩存key會經過⼀系列的update⽅法，udate⽅法由⼀個CacheKey這個對象來執⾏的，這個 update⽅法最終由updateList的list來把五個值存進去，對照上⾯的代碼和下⾯的圖示，你應該能 理解這五個值都是什麼了
![[Pasted image 20211002202901.png]]

這⾥需要注意⼀下最後⼀個值，configuration.getEnvironment().getId()這是什麼，這其實就是 定義在mybatis config.xml中的標籤，⻅如下。
```xml
<environments default="development">
<environment id="development">
<transactionManager type="JDBC"/>
<dataSource type="POOLED">
 <property name="driver" value="${jdbc.driver}"/>
 <property name="url" value="${jdbc.url}"/>
 <property name="username" value="${jdbc.username}"/>
 <property name="password" value="${jdbc.password}"/>
 </dataSource>
</environment>
</environments>
```

那麼我們回歸正題，那麼創建完緩存之後該⽤在何處呢？總不會憑空創建⼀個緩存不使⽤吧？

絕對不會的，經過我們對⼀級緩存的探究之後，我們發現⼀級緩存更多是⽤於查詢操作，畢竟⼀級緩存也叫做查 詢緩存吧，為什麼叫查詢緩存我們⼀會⼉說。我們先來看⼀下這個緩存到底⽤在哪了，我們跟踪到 query⽅法如下：
```java
Override
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds,
ResultHandler resultHandler) throws SQLException {
 BoundSql boundSql = ms.getBoundSql(parameter);
 //创建缓存
 CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
 return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
}
@SuppressWarnings("unchecked")
Override
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds,
ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
 ...
 list = resultHandler == null ? (List<E>) localCache.getObject(key) : null; if (list
!= null) {
 //这个主要是处理存储过程⽤的。
 handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
 } else {
 list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
 }
 ...
}
// queryFromDatabase ⽅法
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds
rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws
SQLException {
 List<E> list;
 localCache.putObject(key, EXECUTION_PLACEHOLDER);
 try {
 list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
 } finally {
 localCache.removeObject(key);
 }
 localCache.putObject(key, list);
 if (ms.getStatementType() == StatementType.CALLABLE) {
localOutputParameterCache.putObject(key, parameter);
 }
 return list;
}
```

如果查不到的話，就從數據庫查，在queryFromDatabase中，會對localcache進⾏寫⼊。 localcache對象的put⽅法最終交給Map進⾏存放
```java
private Map<Object, Object> cache = new HashMap<Object, Object>();
 @Override
 public void putObject(Object key, Object value) { cache.put(key, value);
}
```

## 7.2 ⼆級緩存
⼆級緩存的原理和⼀級緩存原理⼀樣，第⼀次查詢，會將數據放⼊緩存中，然後第⼆次查詢則會直接去 緩存中取。

但是⼀級緩存是基於sqlSession的，⽽⼆級緩存是基於mapper⽂件的namespace的，也 就是說多個sqlSession可以共享⼀個mapper中的⼆級緩存區域，並且如果兩個mapper的namespace 相同，即使是兩個mapper,那麼這兩個mapper中執⾏sql查詢到的數據也將存在相同的⼆級緩存區域中

![[Pasted image 20211002204210.png]]

如何使⽤⼆級緩存

**① 、開啟⼆級緩存**

和⼀級緩存默認開啟不⼀樣，⼆級緩存需要我們⼿動開啟

⾸先在全局配置⽂件sqlMapConfig.xml⽂件中加⼊如下代碼:
```xml
<!--开启⼆级缓存-->
<settings>
 <setting name="cacheEnabled" value="true"/>
</settings>
```
其次在UserMapper.xml⽂件中開啟緩存
```xml
<!--开启⼆级缓存-->
<cache></cache>
```
我們可以看到mapper.xml⽂件中就這麼⼀個空標籤，其實這⾥可以配置,PerpetualCache這個類是 mybatis默認實現緩存功能的類。

我們不寫type就使⽤mybatis默認的緩存，也可以去實現Cache接⼝ 來⾃定義緩存。
![[Pasted image 20211002204409.png]]

```java
public class PerpetualCache implements Cache {
 private final String id;
 private MapcObject, Object> cache = new HashMapC);
 public PerpetualCache(St ring id) { this.id = id;
}
```
我們可以看到⼆級緩存底層還是HashMap結構
```java
public class User implements Serializable(
 //⽤户ID
 private int id;
 //⽤户姓名
 private String username;
 //⽤户性别
 private String sex;
}
```
開啟了⼆級緩存後，還需要將要緩存的pojo實現Serializable接⼝，為了將緩存數據取出執⾏反序列化操 作，因為⼆級緩存數據存儲介質多種多樣，不⼀定只存在內存中，有可能存在硬盤中，如果我們要再取 這個緩存的話，就需要反序列化了。所以mybatis中的pojo都去實現Serializable接⼝

**③、測試**

⼀、測試⼆級緩存和sqlSession⽆關
```java
@Test
public void testTwoCache(){
 //根据 sqlSessionFactory 产⽣ session
 SqlSession sqlSession1 = sessionFactory.openSession();
 SqlSession sqlSession2 = sessionFactory.openSession();
 
 UserMapper userMapper1 = sqlSession1.getMapper(UserMapper. class );
 UserMapper userMapper2 = sqlSession2.getMapper(UserMapper. class );
 //第⼀次查询，发出sql语句，并将查询的结果放⼊缓存中
 User u1 = userMapper1.selectUserByUserId(1);
 System.out.println(u1);
 sqlSession1.close(); //第⼀次查询完后关闭 sqlSession
 
 //第⼆次查询，即使sqlSession1已经关闭了，这次查询依然不发出sql语句
 User u2 = userMapper2.selectUserByUserId(1);
 System.out.println(u2);
 sqlSession2.close();

```
可以看出上⾯兩個不同的sqlSession,第⼀個關閉了，第⼆次查詢依然不發出sql查詢語句

⼆、測試執⾏commit()操作，⼆級緩存數據清空
```java
@Test
public void testTwoCache(){
 //根据 sqlSessionFactory 产⽣ session
 SqlSession sqlSession1 = sessionFactory.openSession();
 SqlSession sqlSession2 = sessionFactory.openSession();
 SqlSession sqlSession3 = sessionFactory.openSession();
 String statement = "com.lagou.pojo.UserMapper.selectUserByUserld" ;
 UserMapper userMapper1 = sqlSession1.getMapper(UserMapper. class );
 UserMapper userMapper2 = sqlSession2.getMapper(UserMapper. class );
 UserMapper userMapper3 = sqlSession2.getMapper(UserMapper. class );
 //第⼀次查询，发出sql语句，并将查询的结果放⼊缓存中
 User u1 = userMapperl.selectUserByUserId( 1 );
 System.out.println(u1);
 sqlSessionl .close(); //第⼀次查询完后关闭sqlSession
 
 //执⾏更新操作，commit()
 u1.setUsername( "aaa" );
 userMapper3.updateUserByUserId(u1);
 sqlSession3.commit();
 
 //第⼆次查询，由于上次更新操作，缓存数据已经清空(防⽌数据脏读)，这⾥必须再次发出sql语
 User u2 = userMapper2.selectUserByUserId( 1 );
 System.out.println(u2);
 sqlSession2.close();
}
```
查看控制台情况：
![[Pasted image 20211002204741.png]]

**④、useCache和flushCache**

mybatis中還可以配置userCache和flushCache等配置項，userCache是⽤來設置是否禁⽤⼆級緩 存的，

在statement中設置useCache=false可以禁⽤當前select語句的⼆級緩存，即每次查詢都會發出 sql去查詢，默認情況是true,即該sql使⽤⼆級緩存

```xml
<select id="selectUserByUserId" useCache="false" resultType="com.lagou.pojo.User" parameterType="int">
 select * from user where id=#{id}
</select>
```
這種情況是針對每次查詢都需要最新的數據sql,要設置成useCache=false，禁⽤⼆級緩存，直接從數 據庫中獲取。

在mapper的同⼀個namespace中，如果有其它insert、update, delete操作數據後需要刷新緩 存，如果不執⾏刷新緩存會出現臟讀。

設置statement配置中的flushCache="true”屬性，默認情況下為true,即刷新緩存，如果改成false則 不會刷新。使⽤緩存時如果⼿動修改數據庫表中的查詢數據會出現臟讀。
```xml
<select id="selectUserByUserId" flushCache="true" useCache="false" resultType="com.lagou.pojo.User" parameterType="int">
 select * from user where id=#{id}
</select>
```
⼀般下執⾏完commit操作都需要刷新緩存，flushCache=true表示刷新緩存，這樣可以避免數據庫臟讀。所以我們不⽤設置，默認即可

## 7.3 二級緩存整合redis
上⾯我們介紹了 mybatis⾃帶的⼆級緩存，但是這個緩存是單服務器⼯作，⽆法實現分佈式緩存。那麼什麼是分佈式緩存呢？

假設現在有兩個服務器1和2,⽤戶訪問的時候訪問了 1服務器，查詢後的緩 存就會放在1服務器上，假設現在有個⽤戶訪問的是2服務器，那麼他在2服務器上就⽆法獲取剛剛那個 緩存，如下圖所示：
![[Pasted image 20211002205207.png]]

為了解決這個問題，就得找⼀個分佈式的緩存，專⻔⽤來存儲緩存數據的，這樣不同的服務器要緩存數據都往它那⾥存，取緩存數據也從它那⾥取，如下圖所示：
![[Pasted image 20211002205254.png]]

如上圖所示，在⼏個不同的服務器之間，我們使⽤第三⽅緩存框架，將緩存都放在這個第三⽅框架中, 然後⽆論有多少台服務器，我們都能從緩存中獲取數據。

這⾥我們介紹mybatis與redis的整合。

剛剛提到過，mybatis提供了⼀個eache接⼝，如果要實現⾃⼰的緩存邏輯，實現cache接⼝開發即可。

mybati s本身默認實現了⼀個，但是這個緩存的實現⽆法實現分佈式緩存，所以我們要⾃⼰來實現。

redis分佈式緩存就可以，mybatis提供了⼀個針對cache接⼝的redis實現類，該類存在mybatis-redis包中實現：

1. pom⽂件
```xml
<dependency>
 <groupId>org.mybatis.caches</groupId>
 <artifactId>mybatis-redis</artifactId>
 <version>1.0.0-beta2</version>
</dependency>
```

2. 配置文件 Mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lagou.mapper.IUserMapper">
	
<cache type="org.mybatis.caches.redis.RedisCache" />
	
<select id="findAll" resultType="com.lagou.pojo.User" useCache="true">
 select * from user
</select>
```

3. redis.properties
```properties
redis.host=localhost
redis.port=6379
redis.connectionTimeout=5000
redis.password=
redis.database=0
```

4. 測試
```java
@Test
public void SecondLevelCache(){
 SqlSession sqlSession1 = sqlSessionFactory.openSession();
 SqlSession sqlSession2 = sqlSessionFactory.openSession();
 SqlSession sqlSession3 = sqlSessionFactory.openSession();
 IUserMapper mapper1 = sqlSession1.getMapper(IUserMapper.class);
 lUserMapper mapper2 = sqlSession2.getMapper(lUserMapper.class);
 lUserMapper mapper3 = sqlSession3.getMapper(IUserMapper.class);
 User user1 = mapper1.findUserById(1);
 sqlSession1.close(); //清空⼀级缓存
 
 User user = new User();
 user.setId(1);
 user.setUsername("lisi");
 mapper3.updateUser(user);
 sqlSession3.commit();
 User user2 = mapper2.findUserById(1);
 System.out.println(user1==user2);
}
```

**原碼分析：**
RedisCache和⼤家普遍實現Mybatis的緩存⽅案⼤同⼩異，⽆⾮是實現Cache接⼝，並使⽤jedis操作緩存；不過該項⽬在設計細節上有⼀些區別；
```java
public final class RedisCache implements Cache {
 public RedisCache(final String id) {
 if (id == null) {
 throw new IllegalArgumentException("Cache instances require anID");
}
 this.id = id;
 RedisConfig redisConfig =
 RedisConfigurationBuilder.getInstance().parseConfiguration();
 pool = new JedisPool(redisConfig, redisConfig.getHost(),
 redisConfig.getPort(),
 redisConfig.getConnectionTimeout(),
 redisConfig.getSoTimeout(), redisConfig.getPassword(), redisConfig.getDatabase(),
redisConfig.getClientName());
}
```
RedisCache在mybatis啟動的時候，由MyBatis的CacheBuilder創建，

創建的⽅式很簡單，就是調⽤ RedisCache的帶有String參數的構造⽅法，即RedisCache(String id)；

⽽在RedisCache的構造⽅法中， 調⽤了 RedisConfigurationBuilder 來創建 RedisConfig 對象，並使⽤ RedisConfig 來創建JedisPool。

RedisConfig類繼承了 JedisPoolConfig，並提供了 host,port等屬性的包裝，簡單看⼀下RedisConfig的 屬性：
```java
public class RedisConfig extends JedisPoolConfig {
private String host = Protocol.DEFAULT_HOST;
private int port = Protocol.DEFAULT_PORT;
private int connectionTimeout = Protocol.DEFAULT_TIMEOUT;
private int soTimeout = Protocol.DEFAULT_TIMEOUT;
private String password;
private int database = Protocol.DEFAULT_DATABASE;
private String clientName;
```
RedisConfig對像是由RedisConfigurationBuilder創建的，簡單看下這個類的主要⽅法：
```java
public RedisConfig parseConfiguration(ClassLoader classLoader) {
 Properties config = new Properties();
 InputStream input =
 classLoader.getResourceAsStream(redisPropertiesFilename);
 if (input != null) {
 try {
 config.load(input);
 } catch (IOException e) {
 throw new RuntimeException(
 "An error occurred while reading classpath property '"
 + redisPropertiesFilename
 + "', see nested exceptions", e);
 } finally {
 try {
 input.close();
 } catch (IOException e) {
 // close quietly
 }
 }
}
 RedisConfig jedisConfig = new RedisConfig();
 setConfigProperties(config, jedisConfig);
 return jedisConfig;
}
```
核⼼的⽅法就是parseConfiguration⽅法，該⽅法從classpath中讀取⼀個redis.properties⽂件:
```properties
host=localhost
port=6379
connectionTimeout=5000
soTimeout=5000
password= database=0 clientName=
```
並將該配置⽂件中的內容設置到RedisConfig對像中，並返回；接下來，就是RedisCache使⽤ RedisConfig類創建完成edisPool；在RedisCache中實現了⼀個簡單的模板⽅法，⽤來操作Redis：
```java
private Object execute(RedisCallback callback) {
 Jedis jedis = pool.getResource();
 try {
 return callback.doWithRedis(jedis);
 } finally {
 jedis.close();
 }
}
```
模板接⼝為RedisCallback，這個接⼝中就只需要實現了⼀個doWithRedis⽅法⽽已：
```java
public interface RedisCallback {
 Object doWithRedis(Jedis jedis);
}
```
接下來看看Cache中最重要的兩個⽅法：putObject和getObject，通過這兩個⽅法來查看mybatis-redis 儲存數據的格式：
```java
@Override
public void putObject(final Object key, final Object value) {
 execute(new RedisCallback() {
 @Override
 public Object doWithRedis(Jedis jedis) {
 jedis.hset(id.toString().getBytes(), key.toString().getBytes(),
SerializeUtil.serialize(value));
 return null;
 }
 });
}
 @Override
 public Object getObject(final Object key) {
 return execute(new RedisCallback() {
 
 @Override
 public Object doWithRedis(Jedis jedis) {
 return SerializeUtil.unserialize(jedis.hget(id.toString().getBytes(),
key.toString().getBytes()));
 }
 });
}
```
可以很清楚的看到，mybatis-redis在存儲數據的時候，是使⽤的hash結構，把cache的id作為這個hash 的key(cache的id在mybatis中就是mapper的namespace)；

這個mapper中的查詢緩存數據作為 hash的field,需要緩存的內容直接使⽤SerializeUtil存儲，SerializeUtil和其他的序列化類差不多，負責 對象的序列化和反序列化；















