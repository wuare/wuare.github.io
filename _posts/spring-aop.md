---
title: Spring AOP学习
date: 2020-08-28 15:10:12
tags: Spring
---
Spring AOP由`AnnotationAwareAspectJAutoProxyCreator.class`类对其它Bean进行代理。
<!-- more -->
`AnnotationAwareAspectJAutoProxyCreator.class`的父类`AbstractAutoProxyCreator`实现`BeanPostProcessor`接口，在Bean实例化的时候会调用`postProcessAfterInitialization`方法，该方法调用`wrapIfNecessary`方法，代码如下：
```java
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
	// ...忽略部分代码
	
	// Create proxy if we have advice.
	Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
	if (specificInterceptors != DO_NOT_PROXY) {
		this.advisedBeans.put(cacheKey, Boolean.TRUE);
		Object proxy = createProxy(
				bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
		this.proxyTypes.put(cacheKey, proxy.getClass());
		return proxy;
	}
}
```
以上方法获取到合法的Advice及Advisor，然后创建代理。
`getAdvicesAndAdvisorsForBean`方法调用`findEligibleAdvisors`方法，代码如下：
```java
protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
	List<Advisor> candidateAdvisors = findCandidateAdvisors(); // 获取所有Advisor
	List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName); // 寻找合法的Advisor
	extendAdvisors(eligibleAdvisors);
	if (!eligibleAdvisors.isEmpty()) {
		eligibleAdvisors = sortAdvisors(eligibleAdvisors);
	}
	return eligibleAdvisors;
}
```
省去一些其它代码，直接查看`findAdvisorsThatCanApply`方法的`canApply`方法，代码如下：
```java
public static boolean canApply(Advisor advisor, Class<?> targetClass, boolean hasIntroductions) {
	if (advisor instanceof IntroductionAdvisor) {
		return ((IntroductionAdvisor) advisor).getClassFilter().matches(targetClass);
	}
	else if (advisor instanceof PointcutAdvisor) {
		PointcutAdvisor pca = (PointcutAdvisor) advisor;
		return canApply(pca.getPointcut(), targetClass, hasIntroductions);
	}
	else {
		// It doesn't have a pointcut so we assume it applies.
		return true;
	}
}
```
如果Advisor是PointcutAdvisor的子类，则调用`canApply(pca.getPointcut(), targetClass, hasIntroductions)`方法，代码如下：
```java
public static boolean canApply(Pointcut pc, Class<?> targetClass, boolean hasIntroductions) {
	if (!pc.getClassFilter().matches(targetClass)) {
		return false;
	}

	MethodMatcher methodMatcher = pc.getMethodMatcher();
	if (methodMatcher == MethodMatcher.TRUE) {
		// No need to iterate the methods if we're matching any method anyway...
		return true;
	}

	IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
	if (methodMatcher instanceof IntroductionAwareMethodMatcher) {
		introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;
	}

	Set<Class<?>> classes = new LinkedHashSet<>();
	if (!Proxy.isProxyClass(targetClass)) {
		classes.add(ClassUtils.getUserClass(targetClass));
	}
	classes.addAll(ClassUtils.getAllInterfacesForClassAsSet(targetClass));

	for (Class<?> clazz : classes) {
		Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
		for (Method method : methods) {
			if (introductionAwareMethodMatcher != null ?
					introductionAwareMethodMatcher.matches(method, targetClass, hasIntroductions) :
					methodMatcher.matches(method, targetClass)) {
				return true;
			}
		}
	}

	return false;
}
```
以上代码先判断Pointcut的classFilter是否能够匹配目标Bean，然后获取MethodMatcher，调用MethodMatcher的matches方法匹配目标Bean的所有接口中的方法。

结合Spring tx模块，查看Spring判断某个Bean是否开始事务的步骤。
查看Spring解析annotation-driven标签的解析器`AnnotationDrivenBeanDefinitionParser`类，parse方法注册了三个类：
* BeanFactoryTransactionAttributeSourceAdvisor.class 该类实现了PointcutAdvisor接口
* AnnotationTransactionAttributeSource.class 
* TransactionInterceptor.class

AnnotationTransactionAttributeSource和TransactionInterceptor是BeanFactoryTransactionAttributeSourceAdvisor的属性
BeanFactoryTransactionAttributeSourceAdvisor的属性pointcut代码如下：
```java
private final TransactionAttributeSourcePointcut pointcut = new TransactionAttributeSourcePointcut() {
	@Override
	@Nullable
	protected TransactionAttributeSource getTransactionAttributeSource() {
		return transactionAttributeSource;
	}
};
```
TransactionAttributeSourcePointcut类的构造方法设置了classFilter，代码如下：
```java
protected TransactionAttributeSourcePointcut() {
	setClassFilter(new TransactionAttributeSourceClassFilter());
}
```

上文分析canApply方法，第一步是获取pointcut的classfilter，调用matches方法，判断目标类是否能进行代理。
Spring tx获取到的classfilter为TransactionAttributeSourceClassFilter，matches方法实现如下：
```java
private class TransactionAttributeSourceClassFilter implements ClassFilter {

	@Override
	public boolean matches(Class<?> clazz) {
		if (TransactionalProxy.class.isAssignableFrom(clazz) ||
				PlatformTransactionManager.class.isAssignableFrom(clazz) ||
				PersistenceExceptionTranslator.class.isAssignableFrom(clazz)) {
			return false;
		}
		TransactionAttributeSource tas = getTransactionAttributeSource();
		return (tas == null || tas.isCandidateClass(clazz));
	}
}
```

`getTransactionAttributeSource()`方法又调用到了BeanFactoryTransactionAttributeSourceAdviso的`private final TransactionAttributeSourcePointcut pointcut = new TransactionAttributeSourcePointcut()`这里，返回初始化类时设置的AnnotationTransactionAttributeSource类实例。

tas.isCandidateClass(clazz)代码如下：
```java
public boolean isCandidateClass(Class<?> targetClass) {
	for (TransactionAnnotationParser parser : this.annotationParsers) {
		if (parser.isCandidateClass(targetClass)) {
			return true;
		}
	}
	return false;
}
```
Spring tx第二步获取MethodMatcher。
TransactionAttributeSourcePointcut的父类StaticMethodMatcherPointcut的getMethodMatcher方法实现为`return this`，所以会调用TransactionAttributeSourcePointcut的matches方法，代码如下：
```java
@Override
public boolean matches(Method method, Class<?> targetClass) {
	TransactionAttributeSource tas = getTransactionAttributeSource();
	return (tas == null || tas.getTransactionAttribute(method, targetClass) != null);
}
```
同上会调用AnnotationTransactionAttributeSource的getTransactionAttribute方法，代码如下：
```java
public TransactionAttribute getTransactionAttribute(Method method, @Nullable Class<?> targetClass) {

	Object cacheKey = getCacheKey(method, targetClass);
	TransactionAttribute cached = this.attributeCache.get(cacheKey);
	if (cached != null) {
		if (cached == NULL_TRANSACTION_ATTRIBUTE) {
			return null;
		}
		else {
			return cached;
		}
	}
	else {
		TransactionAttribute txAttr = computeTransactionAttribute(method, targetClass);
		if (txAttr == null) {
			this.attributeCache.put(cacheKey, NULL_TRANSACTION_ATTRIBUTE);
		}
		else {
			String methodIdentification = ClassUtils.getQualifiedMethodName(method, targetClass);
			if (txAttr instanceof DefaultTransactionAttribute) {
				((DefaultTransactionAttribute) txAttr).setDescriptor(methodIdentification);
			}
			this.attributeCache.put(cacheKey, txAttr);
		}
		return txAttr;
	}
}
```
继续查看computeTransactionAttribute方法：
```java
protected TransactionAttribute computeTransactionAttribute(Method method, @Nullable Class<?> targetClass) {
	// Don't allow no-public methods as required.
	if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
		return null;
	}

	// The method may be on an interface, but we need attributes from the target class.
	// If the target class is null, the method will be unchanged.
	Method specificMethod = AopUtils.getMostSpecificMethod(method, targetClass);

	// First try is the method in the target class.
	TransactionAttribute txAttr = findTransactionAttribute(specificMethod);
	if (txAttr != null) {
		return txAttr;
	}

	// Second try is the transaction attribute on the target class.
	txAttr = findTransactionAttribute(specificMethod.getDeclaringClass());
	if (txAttr != null && ClassUtils.isUserLevelMethod(method)) {
		return txAttr;
	}

	if (specificMethod != method) {
		// Fallback is to look at the original method.
		txAttr = findTransactionAttribute(method);
		if (txAttr != null) {
			return txAttr;
		}
		// Last fallback is the class of the original method.
		txAttr = findTransactionAttribute(method.getDeclaringClass());
		if (txAttr != null && ClassUtils.isUserLevelMethod(method)) {
			return txAttr;
		}
	}

	return null;
}
```
以上方法第一行判断Method是否是public，只有public方法合法。
接下来调用findTransactionAttribute(specificMethod)方法寻找方法上的事务注解，如果找不到，则调用findTransactionAttribute(specificMethod.getDeclaringClass())寻找类上的事务注解，如果找到事务配置，则说明该方法应该被代理。