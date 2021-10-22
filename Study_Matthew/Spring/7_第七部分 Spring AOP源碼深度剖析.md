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

# 7_第七部分： Spring AOP源碼深度剖析

## 第1節 代理對象創建
### 1.1 AOP基礎⽤例準備

Bean定義
```java
@Component
public class LagouBean {
 public void tech(){
 System.out.println("java learning......");
 }
}
```

Aspect定義
```java
package com.lagou;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;
@Component
@Aspect
public class LagouAspect {
 @Pointcut("execution(* com.lagou.*.*(..))")
 public void pointcut(){
 }
 @Before("pointcut()")
 public void before() {
 System.out.println("before method ......");
 }
}
```

測試⽤例
```java
/**
* 测试⽤例：Aop 代理对象创建
*/
@Test
public void testAopProxyBuild(){
 ApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringConfig.class);
 LagouBean lagouBean = applicationContext.getBean(LagouBean.class);
 lagouBean.tech();
}
```

### 1.2 时机点分析
![[Pasted image 20211016153716.png]]

我們發現在 getBean 之前，LagouBean對像已經產⽣（即在第⼀⾏初始化代碼中完成），⽽且該對象是⼀個代理對象（Cglib代理對象），我們斷定，容器初始化過程中⽬標Ban已經完成了代理，返回了代
理對象。

### 1.3 代理对象创建流程 
AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
```java
/**
 *
 * 初始化Bean
 包括Bean后置处理器初始化
 Bean的⼀些初始化⽅法的执⾏init-method
 Bean的实现的声明周期相关接⼝的属性注⼊
 */
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
 // 执⾏所有的AwareMethods
 if (System.getSecurityManager() != null) {
	 AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
	 invokeAwareMethods(beanName, bean);
	 return null;
	 }, getAccessControlContext());
 }
 else {
	 invokeAwareMethods(beanName, bean);
 }
 Object wrappedBean = bean;
 if (mbd == null || !mbd.isSynthetic()) {
	 // 执⾏所有的BeanPostProcessor#postProcessBeforeInitialization 初始化之前的处理器⽅法
	 wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
 }
 try {
	 // 这⾥就开始执⾏afterPropertiesSet（实现了InitializingBean接⼝）⽅法和 initMethod
	 invokeInitMethods(beanName, wrappedBean, mbd);
 }
 catch (Throwable ex) {
	 throw new BeanCreationException(
		 (mbd != null ? mbd.getResourceDescription() : null),
		 beanName, "Invocation of init method failed", ex);
 }
 if (mbd == null || !mbd.isSynthetic()) {
	 // 整个Bean初始化完成，执⾏后置处理器⽅法
	 wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
 }
 return wrappedBean;
}
```

AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization
```java
@Override
 public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName) throws BeansException {
	 Object result = existingBean;
	 // 循环执⾏后置处理器
	 for (BeanPostProcessor processor : getBeanPostProcessors()) {
		 Object current = processor.postProcessAfterInitialization(result, beanName);
		 if (current == null) {
			 return result;
		 }
		 result = current;
	 }
	 return result;
 }
```
![[Pasted image 20211016154139.png]]
創建代理對象的後置處理器AbstractAutoProxyCreator#postProcessAfterInitialization
```java
/**
 * Create a proxy with the configured interceptors if the bean is
 * identified as one to proxy by the subclass.
 * @see #getAdvicesAndAdvisorsForBean
 */
 @Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
 if (bean != null) {
	 // 检查下该类是否已经暴露过了（可能已经创建了，⽐如A依赖B时，创建A时候，就会先去创建B。
	 // 当真正需要创建B时，就没必要再代理⼀次已经代理过的对象）,避免重复创建
	 Object cacheKey = getCacheKey(bean.getClass(), beanName);
	 if (this.earlyProxyReferences.remove(cacheKey) != bean) {
	 	return wrapIfNecessary(bean, beanName, cacheKey);
	 }
 }
 return bean;
}
```
AbstractAutoProxyCreator#wrapIfNecessary
```java
/**
 * Wrap the given bean if necessary, i.e. if it is eligible for being
proxied.
 * @param bean the raw bean instance
 * @param beanName the name of the bean
 * @param cacheKey the cache key for metadata access
 * @return a proxy wrapping the bean, or the raw bean instance as-is
 */
 protected Object wrapIfNecessary(Object bean, String beanName, Object
cacheKey) {
 // targetSourcedBeans包含，说明前⾯创建过
 if (StringUtils.hasLength(beanName) &&
this.targetSourcedBeans.contains(beanName)) {
 return bean;
 }
 if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
 return bean;
 }
 if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(),
beanName)) {
 this.advisedBeans.put(cacheKey, Boolean.FALSE);
 return bean;
 }
 // Create proxy if we have advice.
 // 得到所有候选Advisor，对Advisors和bean的⽅法双层遍历匹配，最终得到⼀个
List<Advisor>，即specificInterceptors
 Object[] specificInterceptors =
getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
 if (specificInterceptors != DO_NOT_PROXY) {
 this.advisedBeans.put(cacheKey, Boolean.TRUE);
 // 重点，创建代理对象
 Object proxy = createProxy(
 bean.getClass(), beanName, specificInterceptors, new
SingletonTargetSource(bean));
 this.proxyTypes.put(cacheKey, proxy.getClass());
 return proxy;
 }
 this.advisedBeans.put(cacheKey, Boolean.FALSE);
 return bean;
 }
```
AbstractAutoProxyCreator#createProxy
```java
/**
 * Create an AOP proxy for the given bean.
 * 为指定 bean 创建代理对象
 */
 protected Object createProxy(Class<?> beanClass, @Nullable String beanName, @Nullable Object[] specificInterceptors, TargetSource targetSource) {
 if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
 	AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory)this.beanFactory, beanName, beanClass);
 }
 // 创建代理的⼯作交给ProxyFactory
 ProxyFactory proxyFactory = new ProxyFactory();
 proxyFactory.copyFrom(this);
 // 根据⼀些情况判断是否要设置proxyTargetClass=true
 if (!proxyFactory.isProxyTargetClass()) {
	 if (shouldProxyTargetClass(beanClass, beanName)) {
		 proxyFactory.setProxyTargetClass(true);
	 }
	 else {
		 evaluateProxyInterfaces(beanClass, proxyFactory);
	 }
 }
 // 把指定和通⽤拦截对象合并, 并都适配成Advisor
 Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
 proxyFactory.addAdvisors(advisors);
 // 设置参数
 proxyFactory.setTargetSource(targetSource);
 customizeProxyFactory(proxyFactory);
 proxyFactory.setFrozen(this.freezeProxy);
 if (advisorsPreFiltered()) {
	 proxyFactory.setPreFiltered(true);
 }
 // 上⾯准备做完就开始创建代理
 return proxyFactory.getProxy(getProxyClassLoader());
 }
```
接著跟進到ProxyFactory中
```java
public class ProxyFactory extends ProxyCreatorSupport {
 public Object getProxy(ClassLoader classLoader) {
 // ⽤ProxyFactory创建AopProxy, 然后⽤AopProxy创建Proxy, 所以这⾥重要的是看获取的 AopProxy
 // 对象是什么,
 // 然后进去看怎么创建动态代理, 提供了两种：jdk proxy, cglib
 return createAopProxy().getProxy(classLoader);
 }
}
public class ProxyCreatorSupport extends AdvisedSupport {
 private AopProxyFactory aopProxyFactory;
 
 public ProxyCreatorSupport() {
 this.aopProxyFactory = new DefaultAopProxyFactory();
 }
 
 protected final synchronized AopProxy createAopProxy() {
 if (!this.active) {
 activate();
 }
 //先获取创建AopProxy的⼯⼚, 再由此创建AopProxy
 return getAopProxyFactory().createAopProxy(this);
 }
 
 public AopProxyFactory getAopProxyFactory() {
 return this.aopProxyFactory;
 }
}
```
流程就是⽤AopProxyFactory創建AopProxy, 再⽤AopProxy創建代理對象，這⾥的AopProxyFactory默
認是DefaultAopProxyFactory，看他的createAopProxy⽅法
```java
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {
 @Override
 public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
	 if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
		 Class<?> targetClass = config.getTargetClass();
		 if (targetClass == null) {
			 throw new AopConfigException("TargetSource cannot determine target class: " + "Either an interface or a target is required for proxy creation.");
		 }
		 if (targetClass.isInterface()) {
			 return new JdkDynamicAopProxy(config);
		 }
			 return new ObjenesisCglibAopProxy(config);
		 } else {
			 return new JdkDynamicAopProxy(config);
		 }
	 }
	 /**
	 * Determine whether the supplied {@link AdvisedSupport} has only the
	 * {@link org.springframework.aop.SpringProxy} interface specified (or no
	 * proxy interfaces specified at all).
	 */
	 private boolean hasNoUserSuppliedProxyInterfaces(AdvisedSupport config) {
	 Class<?>[] interfaces = config.getProxiedInterfaces();
	 return (interfaces.length == 0 || (interfaces.length == 1 && SpringProxy.class.equals(interfaces[0])));
 }
}
```
這⾥決定創建代理對像是⽤JDK Proxy，還是⽤ Cglib 了，最簡單的從使⽤⽅⾯使⽤來說：設置proxyTargetClass=true強制使⽤Cglib 代理，什麼參數都不設並且對像類實現了接⼝則默認⽤JDK 代
理，如果沒有實現接⼝則也必須⽤Cglib

ProxyFactory#getProxy(java.lang.ClassLoader)

------ CglibAopProxy#getProxy(java.lang.ClassLoader)
```java
@Override
 public Object getProxy(@Nullable ClassLoader classLoader) {
	 if (logger.isTraceEnabled()) {
		 logger.trace("Creating CGLIB proxy: " + this.advised.getTargetSource());
	 }
	 try {
		 Class<?> rootClass = this.advised.getTargetClass();
		 Assert.state(rootClass != null, "Target class must be available for creating a CGLIB proxy");
		 Class<?> proxySuperClass = rootClass;
		 if (ClassUtils.isCglibProxyClass(rootClass)) {
			 proxySuperClass = rootClass.getSuperclass();
			 Class<?>[] additionalInterfaces = rootClass.getInterfaces();
			 for (Class<?> additionalInterface : additionalInterfaces) {
				 this.advised.addInterface(additionalInterface);
			 }
		 }
		 // Validate the class, writing log messages as necessary.
		 validateClassIfNecessary(proxySuperClass, classLoader);
		 // 配置 Cglib 增强
		 Enhancer enhancer = createEnhancer();
		 if (classLoader != null) {
		 enhancer.setClassLoader(classLoader);
		 if (classLoader instanceof SmartClassLoader && ((SmartClassLoader) classLoader).isClassReloadable(proxySuperClass)) {
			 enhancer.setUseCache(false);
		 }
		 }
		 enhancer.setSuperclass(proxySuperClass);

		 enhancer.setInterfaces(AopProxyUtils.completeProxiedInterfaces(this.advised));
		 enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
		 enhancer.setStrategy(new ClassLoaderAwareUndeclaredThrowableStrategy(classLoader));
		 Callback[] callbacks = getCallbacks(rootClass);
		 Class<?>[] types = new Class<?>[callbacks.length];
		 for (int x = 0; x < types.length; x++) {
			 types[x] = callbacks[x].getClass();
		 }
		 // fixedInterceptorMap only populated at this point, after getCallbacks call above
		 enhancer.setCallbackFilter(new ProxyCallbackFilter(
			 this.advised.getConfigurationOnlyCopy(), this.fixedInterceptorMap, this.fixedInterceptorOffset));
			 enhancer.setCallbackTypes(types);
		 // ⽣成代理类，并且创建⼀个代理类的实例
		 return createProxyClassAndInstance(enhancer, callbacks);
	 }
	 catch (CodeGenerationException | IllegalArgumentException ex) {
		 throw new AopConfigException("Could not generate CGLIB subclass of " + this.advised.getTargetClass() +
		 ": Common causes of this problem include using a final class or a non-visible class", ex);
	 }
	 catch (Throwable ex) {
		 // TargetSource.getTarget() failed
		 throw new AopConfigException("Unexpected AOP exception", ex);
	 }
}	 
```

**AOP源碼分析類⽅法調⽤關係課堂講解過程中記錄**
```txt
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean
调⽤
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization
调⽤
org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#postProcessAfterInitialization（后置处理器AbstractAutoProxyCreator完成bean代理对象创建）
调⽤
org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#wrapIfNecessary
调⽤
org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#createProxy （在这⼀步把委托对象的aop增强和通⽤拦截进⾏合并，最终给代理对象）
调⽤
org.springframework.aop.framework.DefaultAopProxyFactory#createAopProxy
调⽤
org.springframework.aop.framework.CglibAopProxy#getProxy(java.lang.ClassLoader)
```

## 第2節 Spring聲明式事務控制
聲明式事務很⽅便，尤其純註解模式，僅僅⼏個註解就能控制事務了

思考：這些註解都做了什麼？好神奇！

@EnableTransactionManagement @Transactional

### 2.1 @EnableTransactionManagement
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement {
```
@EnableTransactionManagement 註解使⽤ @Import 標籤引⼊了TransactionManagementConfigurationSelector類，這個類⼜向容器中導⼊了兩個重要的組件
![[Pasted image 20211016155602.png]]

### 2.2 加載事務控制組件
- AutoProxyRegistrar
	
	AutoProxyRegistrar 類的 registerBeanDefinitions ⽅法中⼜註冊了⼀個組件
![[Pasted image 20211016155730.png]]

進⼊ AopConfigUtils.registerAutoProxyCreatorIfNecessary ⽅法
![[Pasted image 20211016160004.png]]

發現最終，註冊了⼀個叫做 InfrastructureAdvisorAutoProxyCreator 的 Bean，⽽這個類是AbstractAutoProxyCreator 的⼦類，實現了 SmartInstantiationAwareBeanPostProcessor 接⼝
```java
public class InfrastructureAdvisorAutoProxyCreator extends AbstractAdvisorAutoProxyCreator
public abstract class AbstractAdvisorAutoProxyCreator extends AbstractAutoProxyCreator
public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport
 implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
```
![[Pasted image 20211016160113.png]]

它實現了SmartInstantiationAwareBeanPostProcessor，說明這是⼀個後置處理器，⽽且跟spring AOP 開啟@EnableAspectJAutoProxy 時註冊的 AnnotationAwareAspectJProxyCreator實現的是同⼀個接⼝，所以說，聲明式事務是 springAOP 思想的⼀種應⽤

- ProxyTransactionManagementConfiguration 組件


```java
/*
* Copyright 2002-2017 the original author or authors.
*
* Licensed under the Apache License, Version 2.0 (the "License");
* you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
*
* https://www.apache.org/licenses/LICENSE-2.0
*
* Unless required by applicable law or agreed to in writing, software
* distributed under the License is distributed on an "AS IS" BASIS,
* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
implied.
* See the License for the specific language governing permissions and
* limitations under the License.
*/
package org.springframework.transaction.annotation;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Role;
import
org.springframework.transaction.config.TransactionManagementConfigUtils;
import
org.springframework.transaction.interceptor.BeanFactoryTransactionAttribut
eSourceAdvisor;
import
org.springframework.transaction.interceptor.TransactionAttributeSource;
import org.springframework.transaction.interceptor.TransactionInterceptor;
/**
* {@code @Configuration} class that registers the Spring infrastructure
beans
* necessary to enable proxy-based annotation-driven transaction
management.
*
* @author Chris Beams
* @since 3.1
* @see EnableTransactionManagement
* @see TransactionManagementConfigurationSelector
*/
@Configuration
public class ProxyTransactionManagementConfiguration extends AbstractTransactionManagementConfiguration {
 @Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
 @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
 public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor(){
 // 事务增强器
 BeanFactoryTransactionAttributeSourceAdvisor advisor = new BeanFactoryTransactionAttributeSourceAdvisor();
 // 向事务增强器中注⼊ 属性解析器 transactionAttributeSource
 advisor.setTransactionAttributeSource(transactionAttributeSource());
 // 向事务增强器中注⼊ 事务拦截器 transactionInterceptor
 advisor.setAdvice(transactionInterceptor());
 if (this.enableTx != null) {
 advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
 }
 return advisor;
 }
 @Bean
 @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
 // 属性解析器 transactionAttributeSource
 public TransactionAttributeSource transactionAttributeSource() {
 return new AnnotationTransactionAttributeSource();
 }
 @Bean
 @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
 // 事务拦截器 transactionInterceptor
 public TransactionInterceptor transactionInterceptor() {
 TransactionInterceptor interceptor = new TransactionInterceptor();
 interceptor.setTransactionAttributeSource(transactionAttributeSource());
 if (this.txManager != null) {
 interceptor.setTransactionManager(this.txManager);
 }
 return interceptor;
 }
}
```

ProxyTransactionManagementConfiguration是⼀個容器配置類，註冊了⼀個組件transactionAdvisor，稱為事務增強器，然後在這個事務增強器中⼜注⼊了兩個屬性：

transactionAttributeSource，即屬性解析器transactionAttributeSource 和 事務攔截器transactionInterceptor

- 屬性解析器 AnnotationTransactionAttributeSource 部分源碼如下
![[Pasted image 20211016160743.png]]

屬性解析器有⼀個成員變量是annotationParsers，是⼀個集合，可以添加多種註解解析器(TransactionAnnotationParser)，我們關注 Spring 的註解解析器，部分源碼如下
![[Pasted image 20211016160826.png]]

屬性解析器的作⽤之⼀就是⽤來解析@Transaction註解

- TransactionInterceptor 事務攔截器，部分源碼如下
![[Pasted image 20211016160909.png]]

![[Pasted image 20211016160928.png]]

- 上述組件如何關聯起來的？
	- 事務攔截器實現了MethodInterceptor接⼝，追溯⼀下上⾯提到的InfrastructureAdvisorAutoProxyCreator後置處理器，它會在代理對象執⾏⽬標⽅法的時候獲取其攔截器鏈，⽽攔截器鏈就是這個TransactionInterceptor，這就把這兩個組件聯繫起來；
	- 構造⽅法傳⼊PlatformTransactionManager(事務管理器)、TransactionAttributeSource(屬性解析器)，但是追溯⼀下上⾯貼的ProxyTransactionManagementConfiguration的源碼，在註冊事務攔截器的時候並沒有調⽤這個帶參構造⽅法，⽽是調⽤的⽆參構造⽅法，然後再調⽤set⽅法注⼊這兩個屬性，效果⼀樣。
- invokeWithinTransaction ⽅法，部分源碼如下（關注1、2、3、4 標註處）
![[Pasted image 20211016161255.png]]

**聲明式事務分析課堂講解過程中記錄**
```txt
@EnableTransactionManagement 註解
1)通過@import引⼊了TransactionManagementConfigurationSelector類
 它的selectImports⽅法導⼊了另外兩個類：AutoProxyRegistrar和 
 
ProxyTransactionManagementConfiguration
2）AutoProxyRegistrar類分析
 ⽅法registerBeanDefinitions中，引⼊了其他類，通過
 AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry)引⼊ InfrastructureAdvisorAutoProxyCreator，
 它繼承了AbstractAutoProxyCreator，是⼀個後置處理器類
3）ProxyTransactionManagementConfiguration 是⼀個添加了@Configuration註解的配置類（註冊bean）
 註冊事務增強器（注⼊屬性解析器、事務攔截器）
 屬性解析器：AnnotationTransactionAttributeSource，內部持有了⼀個解析器集合
 Set<TransactionAnnotationParser> annotationParsers;
 具體使⽤的是SpringTransactionAnnotationParser解析器，⽤來解析@Transactional的事務屬性
 事務攔截器：TransactionInterceptor實現了MethodInterceptor接⼝，該通⽤攔截會在產⽣代理對象之前和aop增強合併，最終⼀起影響到代理對象
 TransactionInterceptor的invoke⽅法中invokeWithinTransaction會觸發原有業務邏輯調⽤（增強事務)
```






























