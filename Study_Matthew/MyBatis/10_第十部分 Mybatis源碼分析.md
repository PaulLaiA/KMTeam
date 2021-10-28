---
界: Framework
門: MyBatis
綱: 
tags: ["#MyBatis","#BQ"]
aliases:
  - 
date: {{DATE:YYYY-MM-DD}}
---



# 第十部分： Mybatis 源碼分析

## 10.1 傳統方式源碼分析

**源码剖析-初始化**
```java
Inputstream inputstream = Resources.getResourceAsStream("mybatis-config.xml");
//这⼀⾏代码正是初始化⼯作的开始。
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
```

进⼊源码分析：
```java
// 1.我们最初调⽤的 build
 public SqlSessionFactory build (InputStream inputStream){
 //调⽤了重载⽅法
 return build(inputStream, null, null);
 }
 
 // 2.调⽤的重载⽅法
 public SqlSessionFactory build (InputStream inputStream, String environment, Properties properties){
 try {
 // XMLConfigBuilder 是专⻔解析 mybatis 的配置⽂件的类
 XMLConfigBuilder parser = new XMLConfigBuilder(inputstream,environment, properties);
 //这⾥⼜调⽤了⼀个重载⽅法。parser.parse()的返回值是 Configuration 对象
 return build(parser.parse());
 } catch (Exception e) {
 throw ExceptionFactory.wrapException("Error building SqlSession.", e)
 }
```

MyBatis 在初始化的時候，會將 MyBatis 的配置信息全部加載到內存中，使⽤ org.apache.ibatis.session.Configuratio n 實例來維護

下⾯進⼊對配置⽂件解析部分：

⾸先對 Configuration 對象進⾏介紹：

	Configuration 對象的結構和 xml 配置⽂件的對象⼏乎相同。
	回顧⼀下 xml 中的配置標籤有哪些：
	properties (屬性)，settings (設置)，typeAliases (類型別名)，typeHandlers (類型處理 器)，
	objectFactory (對象⼯⼚)，mappers (映射器)等 Configuration 也有對應的對象屬性來封裝它們
	也就是說，初始化配置⽂件信息的本質就是創建 Configuration 對象，將解析的 xml 數據封裝到 Configuration 內部屬性中

```java
/**
 * 解析 XML 成 Configuration 对象。
 */
 public Configuration parse () {
	 //若已解析，抛出 BuilderException 异常
	 if (parsed) {
	 	throw new BuilderException("Each XMLConfigBuilder can only be used once.");
	 }
	 //标记已解析
	 parsed = true;
	 // 解析 XML configuration 节点
	 parseConfiguration(parser.evalNode("/configuration"));
	 return configuration;
}
	 /**
	 *解析 XML
	 */
private void parseConfiguration (XNode root){
 try {
	 //issue #117 read properties first
	 // 解析 <properties /> 标签
	 propertiesElement(root.evalNode("properties"));
	 // 解析〈settings /> 标签
	 Properties settings =
	 settingsAsProperties(root.evalNode("settings"));
	 //加载⾃定义的 VFS 实现类
	 loadCustomVfs(settings);
	 // 解析 <typeAliases /> 标签
	 typeAliasesElement(root.evalNode("typeAliases"));
	 //解析 <plugins /> 标签
	 pluginElement(root.evalNode("plugins"));
	 // 解析 <objectFactory /> 标签
	 objectFactoryElement(root.evalNode("objectFactory"));
	 // 解析 <objectWrapperFactory /> 标签
	 objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
	 // 解析 <reflectorFactory /> 标签
	 reflectorFactoryElement(root.evalNode("reflectorFactory"));
	 // 赋值 <settings /> ⾄ Configuration 属性
	 settingsElement(settings);
	 // read it after objectFactory and objectWrapperFactory issue #631
	 // 解析〈environments /> 标签
	 environmentsElement(root.evalNode("environments"));
	 // 解析 <databaseIdProvider /> 标签
	 databaseldProviderElement(root.evalNode("databaseldProvider"));
	 // 解析 <typeHandlers /> 标签
	 typeHandlerElement(root.evalNode("typeHandlers"));
	 //解析 <mappers /> 标签
	 mapperElement(root.evalNode("mappers"));
 } catch (Exception e) {
 	 throw new BuilderException("Error parsing SQL Mapper Configuration.Cause:" + e, e);
 }
}
```

介紹⼀下 MappedStatement ：

作⽤：MappedStatement 與 Mapper 配置⽂件中的⼀個 select/update/insert/delete 節點相對應。

mapper 中配置的標籤都被封裝到了此對像中，主要⽤途是描述⼀條 SQL 語句。

**初始化過程：** 回顧剛開 始介紹的加載配置⽂件的過程中，會對 mybatis-config.xm l 中的各個標籤都進⾏解析，其中有 mappers 標籤⽤來引⼊ mapper.xml ⽂件或者配置 mapper 接⼝的⽬錄。
```xml
<select id="getUser" resultType="user" >
 select * from user where id=#{id}
</select>
```

這樣的⼀個 select 標籤會在初始化配置⽂件時被解析封裝成⼀個 MappedStatement 對象，然後存儲在 Configuration 對象的 mappedStatements 屬性中，

mappedStatements 是⼀個 HashMap，存儲時 key = 全限定類名 + ⽅法名，value = 對應的 MappedStatement 對象。

在 configuration 中對應的屬性為
```java
Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement> ("Mapped Statements collection")
```

在 XMLConfigBuilder 中的處理：
```java
private void parseConfiguration(XNode root) {
	 try {
		 //省略其他标签的处理
		 mapperElement(root.evalNode("mappers"));
	 } catch (Exception e) {
		 throw new BuilderException("Error parsing SQL Mapper Configuration. Cause:" + e, e);
	 }
}
```

到此對 xml 配置⽂件的解析就結束了，回到步驟 2.中調⽤的重載 build ⽅法

```java
// 5.調⽤的重載⽅法
public SqlSessionFactory build(Configuration config) {
 //創建了 DefaultSqlSessionFactory 對象，傳⼊ Configuration 對象。
 return new DefaultSqlSessionFactory(config);
}
```


**源碼剖析-執⾏ SQL 流程**

先簡單介紹 SqlSession ：

SqlSession 是⼀個接⼝，它有兩個實現類：DefaultSqlSession (默認)和

SqlSessionManager (棄⽤，不做介紹)

SqlSession 是 MyBatis 中⽤於和數據庫交互的頂層類，通常將它與 ThreadLocal 綁定，⼀個會話使⽤⼀ 個 SqlSession,並且在使⽤完畢後需要 close
```java
public class DefaultSqlSession implements SqlSession {
	private final Configuration configuration;
	private final Executor executor;
}
```

SqlSession 中的兩個最重要的參數，configuration 與初始化時的相同，Executor 為執⾏器

Executor：

Executor 也是⼀個接⼝，他有三個常⽤的實現類：

BatchExecutor (重⽤語句並執⾏批量更新)

ReuseExecutor (重⽤預處理語句 prepared statements)

SimpleExecutor (普通的執⾏器，默認)

繼續分析，初始化完畢後，我們就要執⾏ SQL 了
```java
SqlSession sqlSession = factory.openSession();
List<User> list = sqlSession.selectList("com.lagou.mapper.UserMapper.getUserByName");
```

獲得 sqlSession
```java
 //6. 进⼊ o penSession ⽅法。
 public SqlSession openSession() {
 //getDefaultExecutorType()传递的是 SimpleExecutor
 	return openSessionFromDataSource(configuration.getDefaultExecutorType(), null,false);
 }

 //7. 进⼊ penSessionFromDataSource。
 //ExecutorType 为 Executor 的类型，TransactionIsolationLevel 为事务隔离级别， autoCommit 是否开启事务
 //openSession 的多个重载⽅法可以指定获得的 SeqSession 的 Executor 类型和事务的处理 
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
 Transaction tx = null;
 try{
	 final Environment environment = configuration.getEnvironment();
	 final TransactionFactory transactionFactory =  getTransactionFactoryFromEnvironment(environment);
	 tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
	 //根据参数创建指定类型的 Executor
	 final Executor executor = configuration.newExecutor(tx, execType);
	 //返回的是 DefaultSqlSession
	 return new DefaultSqlSession(configuration, executor, autoCommit);
 } catch(Exception e){
 closeTransaction(tx); // may have fetched a connection so lets call close()
 }
```

執⾏ sqlsession 中的 api
```java
  //8.进⼊ selectList ⽅法，多个重载⽅法。
 public <E > List < E > selectList(String statement) {
 	return this.selectList(statement, null);
 }
 public <E > List < E > selectList(String statement, Object parameter) {
 	return this.selectList(statement, parameter, RowBounds.DEFAULT);
 }
 public <E > List < E > selectList(String statement, Object parameter, RowBounds rowBounds) {
	 try {
	 //根据传⼊的全限定名 + ⽅法名从映射的 Map 中取出 MappedStatement 对象
	 MappedStatement ms = configuration.getMappedStatement(statement);
	 //调⽤ Executor 中的⽅法处理
	 //RowBounds 是⽤来逻辑分⻚
	 // wrapCollection(parameter)是⽤来装饰集合或者数组参数
	 return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
	 } catch (Exception e) {
	 throw ExceptionFactory.wrapException("Error querying database. Cause: + e, e);
	 } finally {
	 ErrorContext.instance().reset();
 }
```

**源碼剖析-executor**

繼續源碼中的步驟，進⼊ executor.query()
```java
 //此⽅法在 SimpleExecutor 的⽗类 BaseExecutor 中实现
 public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
 //根据传⼊的参数动态获得 SQL 语句，最后返回⽤ BoundSql 对象表示
 BoundSql boundSql = ms.getBoundSql(parameter);
 //为本次查询创建缓存的 Key
 CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
 return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
 }
 //进⼊ query 的重载⽅法中
 public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds,
 ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
 ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
	 if (closed) {
		throw new ExecutorException("Executor was closed.");
	 }
	 if (queryStack == 0 && ms.isFlushCacheRequired()) {
		clearLocalCache();
	 }
	 List<E> list;
	 try {
		 queryStack++;
		 list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
		 if (list != null) {
		 handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
		 } else {
		 //如果缓存中没有本次查找的值，那么从数据库中查询
		 list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
	 }
	 } finally {
		 queryStack--;
	 }
	 if (queryStack == 0) {
	 for (DeferredLoad deferredLoad : deferredLoads) {
		 deferredLoad.load();
		 }
		 // issue #601
		 deferredLoads.clear();
		 if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) { //issue #482 clearLocalCache();
		 }
	 }
	 return list;
 }
 //从数据库查询
 private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
	 List<E> list;
	 localCache.putObject(key, EXECUTION_PLACEHOLDER);
	 try {
		 //查询的⽅法
		 list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
		 } finally {
		 localCache.removeObject(key);
	 }
	 //将查询结果放⼊缓存
	 localCache.putObject(key, list);
	 if (ms.getStatementType() == StatementType.CALLABLE) {
	 	localOutputParameterCache.putObject(key, parameter);
	 }
 	return list;
 }
 // SimpleExecutor 中实现⽗类的 doQuery 抽象⽅法
 public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
	 Statement stmt = null;
	 try {
		 Configuration configuration = ms.getConfiguration();
		 //传⼊参数创建 StatementHanlder 对象来执⾏查询
		 StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
		 //创建 jdbc 中的 statement 对象
		 stmt = prepareStatement(handler, ms.getStatementLog());
		 // StatementHandler 进⾏处理
		 return handler.query(stmt, resultHandler);
	 } finally {
	 	closeStatement(stmt);
	 }
	 }
	 //创建 Statement 的⽅法
	 private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
		 Statement stmt;
		 //条代码中的 getConnection ⽅法经过重重调⽤最后会调⽤ openConnection ⽅法，从连接池中获 得连接。
		 Connection connection = getConnection(statementLog);
		 stmt = handler.prepare(connection, transaction.getTimeout());
		 handler.parameterize(stmt);
	 return stmt;
 }
 //从连接池获得连接的⽅法
 protected void openConnection() throws SQLException {
	 if (log.isDebugEnabled()) {
	 	log.debug("Opening JDBC Connection");
	 }
	 //从连接池获得连接
	 connection = dataSource.getConnection();
	 if (level != null) {
	 	connection.setTransactionIsolation(level.getLevel());
	 }
 }
```

上述的 Executor.query()⽅法⼏經轉折，最後會創建⼀個 StatementHandler 對象，然後將必要的參數傳遞給 StatementHandler，使⽤ StatementHandler 來完成對數據庫的查詢，最終返回 List 結果集。

從上⾯的代碼中我們可以看出，Executor 的功能和作⽤是：

	(1、根據傳遞的參數，完成 SQL 語句的動態解析，⽣成 BoundSql 對象，供 StatementHandler 使⽤；
	(2、為查詢創建緩存，以提⾼性能
	(3、創建 JDBC 的 Statement 連接對象，傳遞給*StatementHandler*對象，返回 List 查詢結果。
	
**源碼剖析-StatementHandler**

StatementHandler 對象主要完成兩個⼯作：

- 對於 JDBC 的 PreparedStatement 類型的對象，創建的過程中，我們使⽤的是 SQL 語句字符串會包含若⼲個？佔位符，我們其後再對占位符進⾏設值。 StatementHandler 通過 parameterize(statement)⽅法對 S tatement 進⾏設值；

- StatementHandler 通過 List query(Statement statement, ResultHandler resultHandler)⽅法來 完成執⾏ Statement，和將 Statement 對象返回的 resultSet 封裝成 List；

進⼊到 StatementHandler 的 parameterize(statement)⽅法的實現：
```java
public void parameterize(Statement statement) throws SQLException {
 //使⽤ ParameterHandler 对象来完成对 Statement 的设值
 parameterHandler.setParameters((PreparedStatement) statement);
}
```

```java
 /** ParameterHandler 类的 setParameters(PreparedStatement ps) 实现
 * 对某⼀个 Statement 进⾏设置参数
 * */
public void setParameters(PreparedStatement ps) throws SQLException {
	ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
	List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
	 if (parameterMappings != null) { 
		 for (int i = 0; i < parameterMappings.size(); i++) { 
			 ParameterMapping parameterMapping = parameterMappings.get(i); 
			 if(parameterMapping.getMode() != ParameterMode.OUT) { 
				 Object value;
				 String propertyName = parameterMapping.getProperty();
				 if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask irst for additional params
					 value = boundSql.getAdditionalParameter(propertyName);
				 } else if (parameterObject == null) { 
					 value = null;
				 } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())){ 
					 value = parameterObject;
				 } else {
					 MetaObject metaObject = configuration.newMetaObject(parameterObject);
					 value = metaObject.getValue(propertyName); 
				 }
				 // 每⼀个 Mapping 都有⼀个 TypeHandler，根据 TypeHandler 来对 preparedStatement 进 ⾏设置参数
				 TypeHandler typeHandler = parameterMapping.getTypeHandler();
				 JdbcType jdbcType = parameterMapping.getJdbcType();
				 if (value == null && jdbcType == null) jdbcType =configuration.getJdbcTypeForNull();
				 //设置参数
				 typeHandler.setParameter(ps, i + 1, value, jdbcType);
			 }
		 }
	 }
 }
```

從上述的代碼可以看到,StatementHandler 的 parameterize(Statement)⽅法調⽤了

ParameterHandler 的 setParameters(statement)⽅法，

ParameterHandler 的 setParameters(Statement )⽅法負責根據我們輸⼊的參數，對 statement 對象的 ?佔位符處進⾏賦值。

進⼊到 StatementHandler 的 List query(Statement statement, ResultHandler resultHandler)⽅法的 實現：
```java
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
 // 1.调⽤ preparedStatemnt。execute()⽅法，然后将 resultSet 交给 ResultSetHandler 处理
 PreparedStatement ps = (PreparedStatement) statement;
 ps.execute();
 
 //2.使⽤ ResultHandler 来处理 ResultSet
 return resultSetHandler.<E> handleResultSets(ps);
}
```

從上述代碼我們可以看出，StatementHandler 的 List query(Statement statement, ResultHandler resultHandler)⽅法的實現，是調⽤了 ResultSetHandler 的 handleResultSets(Statement)⽅法。

ResultSetHandler 的 handleResultSets(Statement)⽅法會將 Statement 語句執⾏後⽣成的 resultSet 結 果集轉換成 List 結果集
```java
public List<Object> handleResultSets(Statement stmt) throws SQLException {
 ErrorContext.instance().activity("handling results").object(mappedStatement.getId());
 //多 ResultSet 的结果集合，每个 ResultSet 对应⼀个 Object 对象。⽽实际上，每 个 Object 是 List<Object> 对象。
 //在不考虑存储过程的多 ResultSet 的情况，普通的查询，实际就⼀个 ResultSet，也 就是说，multipleResults 最多就⼀个元素。
 final List<Object> multipleResults = new ArrayList<>();
 int resultSetCount = 0;
 //获得⾸个 ResultSet 对象，并封装成 ResultSetWrapper 对象
 ResultSetWrapper rsw = getFirstResultSet(stmt);
 //获得 ResultMap 数组
 //在不考虑存储过程的多 ResultSet 的情况，普通的查询，实际就⼀个 ResultSet，也 就是说，resultMaps 就⼀个元素。
 List<ResultMap> resultMaps = mappedStatement.getResultMaps();
 int resultMapCount = resultMaps.size();
 validateResultMapsCount(rsw, resultMapCount); // 校验
 while (rsw != null && resultMapCount > resultSetCount) {
	 //获得 ResultMap 对象
	 ResultMap resultMap = resultMaps.get(resultSetCount);
	 //处理 ResultSet，将结果添加到 multipleResults 中
	 handleResultSet(rsw, resultMap, multipleResults, null);
	 //获得下⼀个 ResultSet 对象，并封装成 ResultSetWrapper 对象
	 rsw = getNextResultSet(stmt);
	 //清理
	 cleanUpAfterHandlingResultSet();
	 // resultSetCount ++
	 resultSetCount++;
 }
 }
 //因为'mappedStatement.resultSets'只在存储过程中使⽤，本系列暂时不考虑，忽略即可
 String[] resultSets = mappedStatement.getResultSets();
 if(resultSets!=null) {
	 while (rsw != null && resultSetCount < resultSets.length) {
	 ResultMapping parentMapping =
	 nextResultMaps.get(resultSets[resultSetCount]);
	 if (parentMapping != null) {
		 String nestedResultMapId = parentMapping.getNestedResultMapId();
		 ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
		 handleResultSet(rsw, resultMap, null, parentMapping);
 	 }
	 rsw = getNextResultSet(stmt);
	 cleanUpAfterHandlingResultSet();
	 resultSetCount++;
 }
 }
 //如果是 multipleResults 单元素，则取⾸元素返回
 return collapseSingleResultList(multipleResults);
}
```

## 10.2 Mapper 代理⽅式：
回顧下寫法:
```java
 public static void main(String[] args) {
	 //前三步都相同
	 InputStream inputStream = Resources.getResourceAsStream("sqlMapConfig.xml");
	 SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
	 SqlSession sqlSession = factory.openSession();
	 //这⾥不再调⽤ SqlSession 的 api,⽽是获得了接⼝对象，调⽤接⼝中的⽅法。
	 UserMapper mapper = sqlSession.getMapper(UserMapper.class);
	 List<User> list = mapper.getUserByName("tom");
 }
```

思考⼀個問題，通常的 Mapper 接⼝我們都沒有實現的⽅法卻可以使⽤，是為什麼呢？

答案很簡單動態 代理開始之前介紹⼀下 MyBatis 初始化時對接⼝的處理：MapperRegistry 是 Configuration 中的⼀個屬性，它內部維護⼀個 HashMap ⽤於存放 mapper 接⼝的⼯⼚類，每個接⼝對應⼀個⼯⼚類。 

mappers 中可以 配置接⼝的包路徑，或者某個具體的接⼝類。

```xml
<mappers>
 <mapper class="com.lagou.mapper.UserMapper"/>
 <package name="com.lagou.mapper"/>
</mappers>
```

當解析 mappers 標籤時，它會判斷解析到的是 mapper 配置⽂件時，會再將對應配置⽂件中的增刪 改查標籤封裝成 MappedStatement 對象，存⼊ mappedStatements 中。

(上⽂介紹了)當判斷解析到接⼝時，會建此接⼝對應的 MapperProxyFactory 對象，存⼊ HashMap 中，key = 接⼝的字節碼對象，value = 此接⼝對應的 MapperProxyFactory 對象。

**源碼剖析-getmapper()**

進⼊ sqlSession.getMapper(UserMapper.class )中
```java
//DefaultSqlSession 中的 getMapper
 public <T> T getMapper(Class<T> type) {
 	return configuration.<T>getMapper(type, this);
 }
 //configuration 中的给 g etMapper
 public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
 	return mapperRegistry.getMapper(type, sqlSession);
 }
 //MapperRegistry 中的 g etMapper
 public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
	 //从 MapperRegistry 中的 HashMap 中拿 MapperProxyFactory
	 final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
	 if (mapperProxyFactory == null) {
	 throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
	 }
	 try {
	 //通过动态代理⼯⼚⽣成示例。
	 return mapperProxyFactory.newInstance(sqlSession);
	 } catch (Exception e) {
	 throw new BindingException("Error getting mapper instance. Cause: " + e, e);
	 }
 }
 //MapperProxyFactory 类中的 newInstance ⽅法
 public T newInstance(SqlSession sqlSession) {
	 //创建了 JDK 动态代理的 Handler 类
	 final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
	 //调⽤了重载⽅法
	 return newInstance(mapperProxy);
 }
 //MapperProxy 类，实现了 InvocationHandler 接⼝
 public class MapperProxy<T> implements InvocationHandler, Serializable {
	 //省略部分源码
	 private final SqlSession sqlSession;
	 private final Class<T> mapperInterface;
	 private final Map<Method, MapperMethod> methodCache;
	 //构造，传⼊了 SqlSession，说明每个 session 中的代理对象的不同的！
	 public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {
		 this.sqlSession = sqlSession;
		 this.mapperInterface = mapperInterface;
		 this.methodCache = methodCache;
 	 }
 //省略部分源码
}
```

**源碼剖析-invoke()**

在動態代理返回了示例後，我們就可以直接調⽤ mapper 類中的⽅法了，但代理對象調⽤⽅法，執⾏是在 MapperProxy 中的 invoke ⽅法中

```java
 public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	 try {
	 //如果是 Object 定义的⽅法，直接调⽤
		 if (Object.class.equals(method.getDeclaringClass())) {
			return method.invoke(this, args);
		 } else if (isDefaultMethod(method)) {
			return invokeDefaultMethod(proxy, method, args);
		 }
	 } catch (Throwable t) {
	 	throw ExceptionUtil.unwrapThrowable(t);
	 }
	 // 获得 MapperMethod 对象
	 final MapperMethod mapperMethod = cachedMapperMethod(method);
	 //重点在这：MapperMethod 最终调⽤了执⾏的⽅法
	 return mapperMethod.execute(sqlSession, args);
 }
```

進⼊ execute ⽅法：
```java
public Object execute(SqlSession sqlSession, Object[] args) {
 Object result;
 //判断 mapper 中的⽅法类型，最终调⽤的还是 SqlSession 中的⽅法 
 switch (command.getType()) {
	 case INSERT: {
		 //转换参数
		 Object param = method.convertArgsToSqlCommandParam(args);
		 //执⾏ INSERT 操作
		 // 转换 rowCount
		 result = rowCountResult(sqlSession.insert(command.getName(), param));
		 break;
	 }
	 case UPdate: {
		 //转换参数
		 Object param = method.convertArgsToSqlCommandParam(args);
		 // 转换 rowCount
		 result = rowCountResult(sqlSession.update(command.getName(), param));
		 break;
	 }
	 case DELETE: {
		 //转换参数
		 Object param = method.convertArgsToSqlCommandParam(args);
		 // 转换 rowCount
		 result = rowCountResult(sqlSession.delete(command.getName(),
		 param));
		 break;
	 }
	 case SELECT:
		 //⽆返回，并且有 ResultHandler ⽅法参数，则将查询的结果，提交给 ResultHandler 进⾏处理
		 if (method.returnsVoid() && method.hasResultHandler()) {
			 executeWithResultHandler(sqlSession, args);
			 result = null;
		 //执⾏查询，返回列表
		 } else if (method.returnsMany()) {
		   result = executeForMany(sqlSession, args);
		 //执⾏查询，返回 Map
		 } else if (method.returnsMap()) {
		   result = executeForMap(sqlSession, args);
		 //执⾏查询，返回 Cursor
		 } else if (method.returnsCursor()) {
		   result = executeForCursor(sqlSession, args);
		 //执⾏查询，返回单个对象
		 } else {
			 //转换参数
			 Object param = method.convertArgsToSqlCommandParam(args);
			 //查询单条
			 result = sqlSession.selectOne(command.getName(), param);
			 if (method.returnsOptional() && (result == null || !method.getReturnType().equals(result.getClass()))) {
				 result = Optional.ofNullable(result);
			 }
		 }
		 break;
	 case FLUSH:
	 	 result = sqlSession.flushStatements();
	 	 break;
	 default:
	 	 throw new BindingException("Unknown execution method for: " + command.getName());
 }
 //返回结果为 null，并且返回类型为基本类型，则抛出 BindingException 异常
 if(result ==null&&method.getReturnType().isPrimitive() &&!method.returnsVoid()) {
	 throw new BindingException("Mapper method '" + command.getName() + " attempted to return null from a method with a primitive
	 return type(" + method.getReturnType() + "). ");
 }
 //返回结果
 return result;
}
```

## 10.3 ⼆级緩存源碼剖析：
⼆級緩存構建在⼀級緩存之上，在收到查詢請求時，MyBatis ⾸先會查詢⼆級緩存，若⼆級緩存未命中，再去查詢⼀級緩存，⼀級緩存沒有，再查詢數據庫。

⼆級緩存------》 ⼀級緩存------》數據庫

與⼀級緩存不同，⼆級緩存和具體的命名空間綁定，⼀個 Mapper 中有⼀個 Cache，相同 Mapper 中的 MappedStatement 共⽤⼀個 Cache，⼀級緩存則是和 SqlSession 綁定。

**啟⽤⼆級緩存**

分為三步⾛：

1）開啟全局⼆級緩存配置：
```xml
<settings>
 <setting name="cacheEnabled" value="true"/>
</settings>
```
2）在需要使⽤⼆級緩存的 Mapper 配置⽂件中配置標籤
```xml
<cache></cache>
```
3）在具體 CURD 標籤上配置 useCache=true
```xml
 <select id="findById" resultType="com.lagou.pojo.User" useCache="true">
 	select * from user where id = #{id}
 </select>
```

**標籤 < cache/> 的解析**

根據之前的 mybatis 源碼剖析，xml 的解析⼯作主要交給 XMLConfigBuilder.parse()⽅法來實現
```java
// XMLConfigBuilder.parse()
public Configuration parse() {
	if (parsed) {
		throw new BuilderException("Each XMLConfigBuilder can only be used once.");
	}
	parsed = true;
	parseConfiguration(parser.evalNode("/configuration"));// 在这⾥
	return configuration;
}
 
// parseConfiguration()
// 既然是在 xml 中添加的，那么我们就直接看关于 mappers 标签的解析
private void parseConfiguration(XNode root) {
 try {
	 Properties settings = settingsAsPropertiess(root.evalNode("settings"));
	 propertiesElement(root.evalNode("properties"));
	 loadCustomVfs(settings);
	 typeAliasesElement(root.evalNode("typeAliases"));
	 pluginElement(root.evalNode("plugins"));
	 objectFactoryElement(root.evalNode("objectFactory"));
	 objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
	 reflectionFactoryElement(root.evalNode("reflectionFactory"));
	 settingsElement(settings);
	 // read it after objectFactory and objectWrapperFactory issue #631
	 environmentsElement(root.evalNode("environments"));
	 databaseIdProviderElement(root.evalNode("databaseIdProvider"));
	 typeHandlerElement(root.evalNode("typeHandlers"));
	 // 就是这⾥
	 mapperElement(root.evalNode("mappers"));
 } catch (Exception e) {
 	throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
 }
}
// mapperElement()
private void mapperElement(XNode parent) throws Exception {
 if (parent != null) {
 	for (XNode child : parent.getChildren()) {
 		if ("package".equals(child.getName())) {
		 String mapperPackage = child.getStringAttribute("name");
		 configuration.addMappers(mapperPackage);
		} else {
			String resource = child.getStringAttribute("resource");
			String url = child.getStringAttribute("url");
			String mapperClass = child.getStringAttribute("class");
			// 按照我们本例的配置，则直接⾛该 if 判断
			if (resource != null && url == null && mapperClass == null) {
				ErrorContext.instance().resource(resource);
				InputStream inputStream = Resources.getResourceAsStream(resource);
				XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
				// ⽣成 XMLMapperBuilder，并执⾏其 parse ⽅法
				mapperParser.parse();
			} else if (resource == null && url != null && mapperClass == null) {
				ErrorContext.instance().resource(url);
				InputStream inputStream = Resources.getUrlAsStream(url);
				XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
				mapperParser.parse();
			} else if (resource == null && url == null && mapperClass != null) {
				Class<?> mapperInterface = Resources.classForName(mapperClass);
				configuration.addMapper(mapperInterface);
			} else {
				throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
			}
		}
	}
 }
}
```

我們來看看解析 Mapper.xml
```java
// XMLMapperBuilder.parse()
public void parse() {
 if (!configuration.isResourceLoaded(resource)) {
	 // 解析 mapper 属性
	 configurationElement(parser.evalNode("/mapper"));
	 configuration.addLoadedResource(resource);
	 bindMapperForNamespace();
 }
 parsePendingResultMaps();
 parsePendingChacheRefs();
 parsePendingStatements();
}
// configurationElement()
private void configurationElement(XNode context) {
 try {
	 String namespace = context.getStringAttribute("namespace");
	 if (namespace == null || namespace.equals("")) {
	 throw new BuilderException("Mapper's namespace cannot be empty");
	 }
	 builderAssistant.setCurrentNamespace(namespace);
	 cacheRefElement(context.evalNode("cache-ref"));
	 // 最终在这⾥看到了关于 cache 属性的处理
	 cacheElement(context.evalNode("cache"));
	 parameterMapElement(context.evalNodes("/mapper/parameterMap"));
	 resultMapElements(context.evalNodes("/mapper/resultMap"));
	 sqlElement(context.evalNodes("/mapper/sql"));
	 // 这⾥会将⽣成的 Cache 包装到对应的 MappedStatement
	 buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
 } catch (Exception e) {
 	 throw new BuilderException("Error parsing Mapper XML. Cause: " + e, e);
 }
}
// cacheElement()
private void cacheElement(XNode context) throws Exception {
 if (context != null) {
 //解析 <cache/> 标签的 type 属性，这⾥我们可以⾃定义 cache 的实现类，⽐如 redisCache，如果没有⾃定义，这⾥使⽤和⼀级缓存相同的 PERPETUAL
 String type = context.getStringAttribute("type", "PERPETUAL");
 Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
 String eviction = context.getStringAttribute("eviction", "LRU");
 Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);
 Long flushInterval = context.getLongAttribute("flushInterval");
 Integer size = context.getIntAttribute("size");
 boolean readWrite = !context.getBooleanAttribute("readOnly", false);
 boolean blocking = context.getBooleanAttribute("blocking", false);
 Properties props = context.getChildrenAsProperties();
 // 构建 Cache 对象
 builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, blocking, props);
 }
}
```

先來看看是如何構建 Cache 對象的

**MapperBuilderAssistant.useNewCache()**
```java
public Cache useNewCache(Class<? extends Cache> typeClass,
 Class<? extends Cache> evictionClass,
 Long flushInterval,
 Integer size,
 boolean readWrite,
 boolean blocking,
 Properties props) {
	 // 1.⽣成 Cache 对象
	 Cache cache = new CacheBuilder(currentNamespace)
	 //这⾥如果我们定义了 <cache/> 中的 type，就使⽤⾃定义的 Cache,否则使⽤和⼀级缓存相同的 PerpetualCache
	 .implementation(valueOrDefault(typeClass, PerpetualCache.class))
	 .addDecorator(valueOrDefault(evictionClass, LruCache.class))
	 .clearInterval(flushInterval)
	 .size(size)
	 .readWrite(readWrite)
	 .blocking(blocking)
	 .properties(props)
	 .build();
	 // 2.添加到 Configuration 中
	 configuration.addCache(cache);
	 // 3.并将 cache 赋值给 MapperBuilderAssistant.currentCache
	 currentCache = cache;
	 return cache;
}
```

我們看到⼀個 Mapper.xml 只會解析⼀次標籤，也就是只創建⼀次 Cache 對象，放進 configuration 中，並將 cache 賦值給 MapperBuilderAssistant.currentCache

**buildStatementFromContext(context.evalNodes("select|insert|update|delete"));將 Cache 包裝到 MappedStatement**
```java
// buildStatementFromContext()
private void buildStatementFromContext(List<XNode> list) {
 if (configuration.getDatabaseId() != null) {
 	buildStatementFromContext(list, configuration.getDatabaseId());
 }
 buildStatementFromContext(list, null);
}
//buildStatementFromContext()
private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
 for (XNode context : list) {
	 final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
	 try {
		// 每⼀条执⾏语句转换成⼀个 MappedStatement
		statementParser.parseStatementNode();
	 } catch (IncompleteElementException e) {
		configuration.addIncompleteStatement(statementParser);
	 }
 }
}
// XMLStatementBuilder.parseStatementNode();
public void parseStatementNode() {
 String id = context.getStringAttribute("id");
 String databaseId = context.getStringAttribute("databaseId");
 ...
 Integer fetchSize = context.getIntAttribute("fetchSize");
 Integer timeout = context.getIntAttribute("timeout");
 String parameterMap = context.getStringAttribute("parameterMap");
 String parameterType = context.getStringAttribute("parameterType");
 Class<?> parameterTypeClass = resolveClass(parameterType);
 String resultMap = context.getStringAttribute("resultMap");
 String resultType = context.getStringAttribute("resultType");
 String lang = context.getStringAttribute("lang");
 LanguageDriver langDriver = getLanguageDriver(lang);
 ...
 // 创建 MappedStatement 对象
 builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
 fetchSize, timeout, parameterMap,parameterTypeClass, resultMap, resultTypeClass, resultSetTypeEnum, flushCache, useCache,resultOrdered,
 keyGenerator, keyProperty, keyColumn,databaseId, langDriver, resultSets);
}
// builderAssistant.addMappedStatement()
public MappedStatement addMappedStatement(
 String id,
 ...) {
 if (unresolvedCacheRef) {
 	throw new IncompleteElementException("Cache-ref not yet resolved");
 }
 id = applyCurrentNamespace(id, false);
 boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
 //创建 MappedStatement 对象
 MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
 ...
 .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
 .useCache(valueOrDefault(useCache, isSelect))
 .cache(currentCache);// 在这⾥将之前⽣成的 Cache 封装到 MappedStatement
 ParameterMap statementParameterMap = getStatementParameterMap(parameterMap, parameterType, id);
 if (statementParameterMap != null) {
 	statementBuilder.parameterMap(statementParameterMap);
 }
 MappedStatement statement = statementBuilder.build();
 configuration.addMappedStatement(statement);
 return statement;
}
```

我們看到將 Mapper 中創建的 Cache 對象，加⼊到了每個 MappedStatement 對像中，也就是同⼀個 Mapper 中所有的 2

有關於標籤的解析就到這了。

**查詢源碼分析**

**CachingExecutor**
```java
// CachingExecutor
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
 BoundSql boundSql = ms.getBoundSql(parameterObject);
 // 创建 CacheKey
 CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
 return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
 // 从 MappedStatement 中获取 Cache，注意这⾥的 Cache 是从 MappedStatement 中获取的
 // 也就是我们上⾯解析 Mapper 中 <cache/> 标签中创建的，它保存在 Configration 中
 // 我们在上⾯解析 blog.xml 时分析过每⼀个 MappedStatement 都有⼀个 Cache 对象，就是这⾥
 Cache cache = ms.getCache();
 // 如果配置⽂件中没有配置 <cache>，则 cache 为空
 if (cache != null) {
	  //如果需要刷新缓存的话就刷新：flushCache="true"
	 flushCacheIfRequired(ms);
	 if (ms.isUseCache() && resultHandler == null) {
		 ensureNoOutParams(ms, boundSql);
		 // 访问⼆级缓存
		 List<E> list = (List<E>) tcm.getObject(cache, key);
		 // 缓存未命中
		 if (list == null) {
			 // 如果没有值，则执⾏查询，这个查询实际也是先⾛⼀级缓存查询，⼀级缓存也没有的话，则进⾏ DB 查询
			 list = delegate.<E>query(ms, parameterObject, rowBounds, resultHandler,key, boundSql);
			 // 缓存查询结果
			 tcm.putObject(cache, key, list);
		 }
		 return list;
	 }
 }
 return delegate.<E>query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```
如果設置了 flushCache="true"，則每次查詢都會刷新緩存
```xml
<!-- 执⾏此语句清空缓存 -->
<select id="findbyId" resultType="com.lagou.pojo.user" useCache="true"
flushCache="true" >
 select * from t_demo
</select>
```
如上，注意⼆級緩存是從 MappedStatement 中獲取的。由於 MappedStatement 存在於全局配置中，可以多個 CachingExecutor 獲取到，這樣就會出現線程安全問題。

除此之外，若不加以控制，多個事務共⽤⼀個緩存實例，會導致臟讀問題。 ⾄於臟讀問題，需要藉助其他類來處理，也就是上⾯代碼中 tcm 變量對應的類型。

下⾯分析⼀下。

**TransactionalCacheManager**
```java
/** 事务缓存管理器 */
public class TransactionalCacheManager {
 // Cache 与 TransactionalCache 的映射关系表
 private final Map<Cache, TransactionalCache> transactionalCaches = new HashMap<Cache, TransactionalCache>();
 public void clear(Cache cache) {
	 // 获取 TransactionalCache 对象，并调⽤该对象的 clear ⽅法，下同
	 getTransactionalCache(cache).clear();
 }
 public Object getObject(Cache cache, CacheKey key) {
	 // 直接从 TransactionalCache 中获取缓存
	 return getTransactionalCache(cache).getObject(key);
 }
 public void putObject(Cache cache, CacheKey key, Object value) {
	 // 直接存⼊ TransactionalCache 的缓存中
	 getTransactionalCache(cache).putObject(key, value);
 }
 public void commit() {
	 for (TransactionalCache txCache : transactionalCaches.values()) {
	 	txCache.commit();
	 }
 }
 public void rollback() {
	 for (TransactionalCache txCache : transactionalCaches.values()) {
	 	txCache.rollback();
	 }
 }
 private TransactionalCache getTransactionalCache(Cache cache) {
	 // 从映射表中获取 TransactionalCache
	 TransactionalCache txCache = transactionalCaches.get(cache);
	 if (txCache == null) {
		 // TransactionalCache 也是⼀种装饰类，为 Cache 增加事务功能
		 // 创建⼀个新的 TransactionalCache，并将真正的 Cache 对象存进去
		 txCache = new TransactionalCache(cache);
		 transactionalCaches.put(cache, txCache);
	 }
	 return txCache;
 }
}
```

TransactionalCacheManager 內部維護了 Cache 實例與 TransactionalCache 實例間的映射關係，該類也僅負責維護兩者的映射關係，真正做事的還是 TransactionalCache。 

TransactionalCache 是⼀種緩存裝飾器，可以為 Cache 實例增加事務功能。

我在之前提到的髒讀問題正是由該類進⾏處理的。下⾯分析⼀下該類的邏輯。

**TransactionalCache**
```java
public class TransactionalCache implements Cache {
 //真正的缓存对象，和上⾯的 Map<Cache, TransactionalCache> 中的 Cache 是同⼀个
 private final Cache delegate;
 private boolean clearOnCommit;
 // 在事务被提交前，所有从数据库中查询的结果将缓存在此集合中
 private final Map<Object, Object> entriesToAddOnCommit;
 // 在事务被提交前，当缓存未命中时，CacheKey 将会被存储在此集合中
 private final Set<Object> entriesMissedInCache;
 @Override
 public Object getObject(Object key) {
	 // 查询的时候是直接从 delegate 中去查询的，也就是从真正的缓存对象中查询
	 Object object = delegate.getObject(key);
	 if (object == null) {
		 // 缓存未命中，则将 key 存⼊到 entriesMissedInCache 中
		 entriesMissedInCache.add(key);
	 }
	 if (clearOnCommit) {
	 	return null;
	 } else {
	 	return object;
	 }
 }
 @Override
 public void putObject(Object key, Object object) {
	 // 将键值对存⼊到 entriesToAddOnCommit 这个 Map 中中，⽽⾮真实的缓存对象 delegate 中
	 entriesToAddOnCommit.put(key, object);
 }
 @Override
 public Object removeObject(Object key) {
 	return null;
 }
 @Override
 public void clear() {
	 clearOnCommit = true;
	 // 清空 entriesToAddOnCommit，但不清空 delegate 缓存
	 entriesToAddOnCommit.clear();
 }
 public void commit() {
	 // 根据 clearOnCommit 的值决定是否清空 delegate
	 if (clearOnCommit) {
	 	delegate.clear();
 	 }
	 // 刷新未缓存的结果到 delegate 缓存中
	 flushPendingEntries();
	 // 重置 entriesToAddOnCommit 和 entriesMissedInCache
	 reset();
 }
 public void rollback() {
	 unlockMissedEntries();
	 reset();
 }
 private void reset() {
	 clearOnCommit = false;
	 // 清空集合
	 entriesToAddOnCommit.clear();
	 entriesMissedInCache.clear();
 }
 private void flushPendingEntries() {
	 for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
		 // 将 entriesToAddOnCommit 中的内容转存到 delegate 中
		 delegate.putObject(entry.getKey(), entry.getValue());
	 }
	 for (Object entry : entriesMissedInCache) {
		 if (!entriesToAddOnCommit.containsKey(entry)) {
		 // 存⼊空值
		 delegate.putObject(entry, null);
		 }
	 }
 }
 private void unlockMissedEntries() {
	 for (Object entry : entriesMissedInCache) {
		 try {
			 // 调⽤ removeObject 进⾏解锁
			 delegate.removeObject(entry);
		 } catch (Exception e) {
		 	 log.warn("...");
		 }
	 }
 }
}
```
存儲⼆級緩存對象的時候是放到了 TransactionalCache.entriesToAddOnCommit 這個 map 中，但是每次查詢的時候是直接從 TransactionalCache.delegate 中去查詢的，

所以這個⼆級緩存查詢數據庫後，設置緩存值是沒有⽴刻⽣效的，主要是因為直接存到 delegate 會導致臟數據問題

**為何只有 SqlSession 提交或關閉之後？**

那我們來看下 SqlSession.commit()⽅法做了什麼

SqlSession
```java
@Override
public void commit(boolean force) {
 try {
	 // 主要是这句
	 executor.commit(isCommitOrRollbackRequired(force));
	 dirty = false;
 } catch (Exception e) {
 	 throw ExceptionFactory.wrapException("Error committing transaction. Cause: " + e, e);
 } finally {
 	 ErrorContext.instance().reset();
 }
}
// CachingExecutor.commit()
@Override
public void commit(boolean required) throws SQLException {
 delegate.commit(required);
 tcm.commit();// 在这⾥
}
// TransactionalCacheManager.commit()
public void commit() {
 for (TransactionalCache txCache : transactionalCaches.values()) {
 	 txCache.commit();// 在这⾥
 }
}
// TransactionalCache.commit()
public void commit() {
 if (clearOnCommit) {
 	 delegate.clear();
 }
 flushPendingEntries();//这⼀句
 reset();
}
// TransactionalCache.flushPendingEntries()
private void flushPendingEntries() {
 for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
	 // 在这⾥真正的将 entriesToAddOnCommit 的对象逐个添加到 delegate 中，只有这时，⼆级缓存才真正的⽣效
	 delegate.putObject(entry.getKey(), entry.getValue());
 }
 for (Object entry : entriesMissedInCache) {
	 if (!entriesToAddOnCommit.containsKey(entry)) {
	 	 delegate.putObject(entry, null);
	 }
 }
}
```

**⼆級緩存的刷新**

我們來看看 SqlSession 的更新操作
```java
public int update(String statement, Object parameter) {
 int var4;
 try {
	 this.dirty = true;
	 MappedStatement ms = this.configuration.getMappedStatement(statement);
	 var4 = this.executor.update(ms, this.wrapCollection(parameter));
 } catch (Exception var8) {
 	 throw ExceptionFactory.wrapException("Error updating database. Cause: " + var8, var8);
 } finally {
 	 ErrorContext.instance().reset();
 }
 return var4;
}
public int update(MappedStatement ms, Object parameterObject) throws SQLException {
 this.flushCacheIfRequired(ms);
 return this.delegate.update(ms, parameterObject);
}
private void flushCacheIfRequired(MappedStatement ms) {
 //获取 MappedStatement 对应的 Cache，进⾏清空
 Cache cache = ms.getCache();
 //SQL 需设置 flushCache="true" 才会执⾏清空
 if (cache != null && ms.isFlushCacheRequired()) {
 	 this.tcm.clear(cache);
 }
}
```

MyBatis ⼆級緩存只適⽤於不常進⾏增、刪、改的數據，⽐如國家⾏政區省市區街道數據。 ⼀但數據變更，MyBatis 會清空緩存。因此⼆級緩存不適⽤於經常進⾏更新的數據。

**總結：**

在⼆級緩存的設計上，MyBatis ⼤量地運⽤了裝飾者模式，如 CachingExecutor, 以及各種 Cache 接⼝的裝飾器。

- ⼆級緩存實現了 Sqlsession 之間的緩存數據共享，屬於 namespace 級別
- ⼆級緩存具有豐富的緩存策略。
- ⼆級緩存可由多個裝飾器，與基礎緩存組合⽽成
- ⼆級緩存⼯作由 ⼀個緩存裝飾執⾏器 CachingExecutor 和 ⼀個事務型預緩存 TransactionalCache 完成。


  
## 10.4 延遲加載源碼剖析：

**什么是延遲加載？**

**問題**

 在開發過程中很多時候我們並不需要總是在加載⽤戶信息時就⼀定要加載他的訂單信息。此時就是我們所說的延遲加載。
 
舉個栗⼦

	在⼀對多中，當我們有⼀個⽤戶，它有個 100 個訂單
	 在查詢⽤戶的時候，要不要把關聯的訂單查出來？
	 在查詢訂單的時候，要不要把關聯的⽤戶查出來？
	 
	回答
	 在查詢⽤戶時，⽤戶下的訂單應該是，什麼時候⽤，什麼時候查詢。
	 在查詢訂單時，訂單所屬的⽤戶信息應該是隨著訂單⼀起查詢出來。
	 
	 
 **延遲加載**
就是在需要⽤到數據時才進⾏加載，不需要⽤到數據時就不加載數據。延遲加載也稱懶加載。

* 優點：
 先從單表查詢，需要時再從關聯表去關聯查詢，⼤⼤提⾼數據庫性能，因為查詢單表要⽐關聯查詢多張表速度要快。
 
* 缺點：
 因為只有當需要⽤到數據時，才會進⾏數據庫查詢，這樣在⼤批量數據查詢時，因為查詢⼯作也要消耗時間，所以可
能造成⽤戶等待時間變⻓，造成⽤戶體驗下降。
 
* 在多表中：
 ⼀對多，多對多：通常情況下採⽤延遲加載
 ⼀對⼀（多對⼀）：通常情況下採⽤⽴即加載
 
* 注意：
 延遲加載是基於嵌套查詢來實現的

**實現**

**局部延遲加載**

在 association 和 collection 標籤中都有⼀個 fetchType 屬性，通過修改它的值，可以修改局部的加載策略。
```xml
<!-- 开启⼀对多 延迟加载 -->
<resultMap id="userMap" type="user">
 <id column="id" property="id"></id>
 <result column="username" property="username"></result>
 <result column="password" property="password"></result>
 <result column="birthday" property="birthday"></result>
 <!--
 fetchType="lazy" 懒加载策略
 fetchType="eager" ⽴即加载策略
 -->
 <collection property="orderList" ofType="order" column="id" 
 select="com.lagou.dao.OrderMapper.findByUid" fetchType="lazy">
 </collection>
</resultMap>
<select id="findAll" resultMap="userMap">
 SELECT * FROM `user`
</select>
```

**全局延遲加載**

在 Mybatis 的核⼼配置⽂件中可以使⽤ setting 標籤修改全局的加載策略。
```xml
<settings>
 <!--开启全局延迟加载功能-->
 <setting name="lazyLoadingEnabled" value="true"/>
</settings>
```

**注意**
```xml
<!-- 关闭⼀对⼀ 延迟加载 -->
<resultMap id="orderMap" type="order">
 <id column="id" property="id"></id>
 <result column="ordertime" property="ordertime"></result>
 <result column="total" property="total"></result>
 <!--
 fetchType="lazy" 懒加载策略
 fetchType="eager" ⽴即加载策略
 -->
 <association property="user" column="uid" javaType="user"
 select="com.lagou.dao.UserMapper.findById" fetchType="eager"> 
</association>
</resultMap>
<select id="findAll" resultMap="orderMap">
 SELECT * from orders
</select>

```

**延遲加載原理實現**
它的原理是，使⽤ CGLIB 或 Javassist( 默認 ) 創建⽬標對象的代理對象。當調⽤代理對象的延遲加載屬性的 getting ⽅法時，進⼊攔截器⽅法。

⽐如調⽤ a.getB().getName() ⽅法，進⼊攔截器的 invoke(...) ⽅法，發現 a.getB() 需要延遲加載時，那麼就會單獨發送事先保存好的查詢關聯 B 對象的 SQL ，把 B 查詢上來，然後調⽤ a.setB(b) ⽅法，於是 a 對象 b 屬性就有值了，接著完成 a.getB().getName() ⽅法的調⽤。這就是延遲加載的基本原理

總結：延遲加載主要是通過動態代理的形式實現，通過代理攔截到指定⽅法，執⾏數據加載。

**延遲加載原理（源碼剖析)**

MyBatis 延遲加載主要使⽤：Javassist，Cglib 實現，類圖展示：
![[Pasted image 20211003162023.png]]

**Setting 配置加载：**
```java
public class Configuration {
/** aggressiveLazyLoading：
 * 當開啟時，任何⽅法的調⽤都會加載該對象的所有屬性。否則，每個屬性會按需加載（參考 lazyLoadTriggerMethods).
 * 默認為 true
 * */
 protected boolean aggressiveLazyLoading;
 /**
 * 延遲加載觸發⽅法
 */
 protected Set<String> lazyLoadTriggerMethods = new HashSet<String>(Arrays.asList(new String[] { "equals", "clone", "hashCode", "toString" }));
 /** 是否開啟延遲加載 */
 protected boolean lazyLoadingEnabled = false;
 
 /**
 * 默認使⽤ Javassist 代理⼯⼚
 * @param proxyFactory
 */
 public void setProxyFactory(ProxyFactory proxyFactory) {
	 if (proxyFactory == null) {
	 	 proxyFactory = new JavassistProxyFactory();
	 }
	 this.proxyFactory = proxyFactory;
 }
 
 //省略...
}
```

**延遲加載代理對象創建**

 Mybatis 的查詢結果是由 ResultSetHandler 接⼝的 handleResultSets()⽅法處理的。 
 
 ResultSetHandler 接⼝只有⼀個實現，DefaultResultSetHandler，接下來看下延遲加載相關的⼀個核⼼的⽅法
```java
<code class="language-Java">//#mark 创建结果对象
 private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {
	 this.useConstructorMappings = false; // reset previous mapping result
	 final List&lt;Class&lt;?&gt;&gt; constructorArgTypes = new ArrayList&lt;Class&lt;?&gt;&gt;();
	 final List&lt;Object&gt; constructorArgs = new ArrayList&lt;Object&gt;();
	 //#mark 创建返回的结果映射的真实对象
	 Object resultObject = createResultObject(rsw, resultMap, constructorArgTypes, constructorArgs, columnPrefix);
	 if (resultObject != null &amp;&amp; !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
		  final List&lt;ResultMapping&gt; propertyMappings = resultMap.getPropertyResultMappings();
		  for (ResultMapping propertyMapping : propertyMappings) {
				 // 判断属性有没配置嵌套查询，如果有就创建代理对象
				 if (propertyMapping.getNestedQueryId() != null &amp;&amp; propertyMapping.isLazy()) {
					  //#mark 创建延迟加载代理对象
					  resultObject = configuration.getProxyFactory().createProxy(resultObject, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
					  break;
					  }
				 }
		  }
	 this.useConstructorMappings = resultObject != null &amp;&amp; !constructorArgTypes.isEmpty(); // set current mapping result
	 return resultObject;
 }
```
默認採⽤ javassistProxy 進⾏代理對象的創建
![[Pasted image 20211003162537.png]]

JavasisstProxyFactory 實現
```java
public class JavassistProxyFactory implements
org.apache.ibatis.executor.loader.ProxyFactory {
 
 /**
 * 接⼝实现
 * @param target ⽬标结果对象
 * @param lazyLoader 延迟加载对象
 * @param configuration 配置
 * @param objectFactory 对象⼯⼚
 * @param constructorArgTypes 构造参数类型
 * @param constructorArgs 构造参数值
 * @return
 */
 @Override
 public Object createProxy(Object target, ResultLoaderMap lazyLoader, Configuration
configuration, ObjectFactory objectFactory, List&lt;Class&lt;?&gt;&gt;
constructorArgTypes, List&lt;Object&gt; constructorArgs) {
 return EnhancedResultObjectProxyImpl.createProxy(target, lazyLoader, configuration,
objectFactory, constructorArgTypes, constructorArgs);
 }
 //省略...
 
 /**
 * 代理对象实现，核⼼逻辑执⾏
 */
 private static class EnhancedResultObjectProxyImpl implements MethodHandler {
 
 /**
 * 创建代理对象
 * @param type
 * @param callback
 * @param constructorArgTypes
 * @param constructorArgs
 * @return
 */
 static Object crateProxy(Class&lt;?&gt; type, MethodHandler callback,
List&lt;Class&lt;?&gt;&gt; constructorArgTypes, List&lt;Object&gt; constructorArgs) {
 ProxyFactory enhancer = new ProxyFactory();
 enhancer.setSuperclass(type);
 try {
 //通过获取对象⽅法，判断是否存在该⽅法
 type.getDeclaredMethod(WRITE_REPLACE_METHOD);
 // ObjectOutputStream will call writeReplace of objects returned by writeReplace
 if (log.isDebugEnabled()) {
 log.debug(WRITE_REPLACE_METHOD + &quot; method was found on bean &quot; + type
+ &quot;, make sure it returns this&quot;);
 }
 } catch (NoSuchMethodException e) {
 //没找到该⽅法，实现接⼝
 enhancer.setInterfaces(new Class[]{WriteReplaceInterface.class});
 } catch (SecurityException e) {
 // nothing to do here
 }
 Object enhanced;
 Class&lt;?&gt;[] typesArray = constructorArgTypes.toArray(new
Class[constructorArgTypes.size()]);
 Object[] valuesArray = constructorArgs.toArray(new Object[constructorArgs.size()]);
 try {
 //创建新的代理对象
 enhanced = enhancer.create(typesArray, valuesArray);
 } catch (Exception e) {
 throw new ExecutorException(&quot;Error creating lazy proxy. Cause: &quot; + e,
e);
 }
 //设置代理执⾏器
 ((Proxy) enhanced).setHandler(callback);
 return enhanced;
 }
 
 
 /**
 * 代理对象执⾏
 * @param enhanced 原对象
 * @param method 原对象⽅法
 * @param methodProxy 代理⽅法
 * @param args ⽅法参数
 * @return
 * @throws Throwable
 */
 @Override
 public Object invoke(Object enhanced, Method method, Method methodProxy, Object[]
args) throws Throwable {
 final String methodName = method.getName();
 try {
 synchronized (lazyLoader) {
 if (WRITE_REPLACE_METHOD.equals(methodName)) {
 //忽略暂未找到具体作⽤
 Object original;
 if (constructorArgTypes.isEmpty()) {
 original = objectFactory.create(type);
 } else {
 original = objectFactory.create(type, constructorArgTypes,
constructorArgs);
 }
 PropertyCopier.copyBeanProperties(type, enhanced, original);
 if (lazyLoader.size() &gt; 0) {
 return new JavassistSerialStateHolder(original,
lazyLoader.getProperties(), objectFactory, constructorArgTypes, constructorArgs);
 } else {
 return original;
 }
 } else {
 //延迟加载数量⼤于 0
 if (lazyLoader.size() &gt; 0 &amp;&amp;
!FINALIZE_METHOD.equals(methodName)) {
 //aggressive ⼀次加载性所有需要要延迟加载属性或者包含触发延迟加载⽅法
 if (aggressive || lazyLoadTriggerMethods.contains(methodName)) {
 log.debug(&quot;==&gt; laze lod trigger method:&quot; + methodName +
&quot;,proxy method:&quot; + methodProxy.getName() + &quot; class:&quot; +
enhanced.getClass());
 //⼀次全部加载
 lazyLoader.loadAll();
 } else if (PropertyNamer.isSetter(methodName)) {
 //判断是否为 set ⽅法，set ⽅法不需要延迟加载
 final String property = PropertyNamer.methodToProperty(methodName);
 lazyLoader.remove(property);
 } else if (PropertyNamer.isGetter(methodName)) {
 final String property = PropertyNamer.methodToProperty(methodName);
 if (lazyLoader.hasLoader(property)) {
 //延迟加载单个属性
 lazyLoader.load(property);
 log.debug(&quot;load one :&quot; + methodName);
 }
 }
 }
 }
 }
 return methodProxy.invoke(enhanced, args);
 } catch (Throwable t) {
 throw ExceptionUtil.unwrapThrowable(t);
 }
 }
 }
```

**注意事項**

IDEA 調試問題 

	當配置 aggressiveLazyLoading=true，在使⽤ IDEA 進⾏調試的時候，如果斷點打到代理執⾏邏輯當中，你會發現延遲加載的代碼永遠都不能進⼊，總是會被提前執⾏。
	主要產⽣的原因在 aggressiveLazyLoading，因為在調試的時候，IDEA 的 Debuger 窗體中已經觸發了延遲加載對象的⽅法。



