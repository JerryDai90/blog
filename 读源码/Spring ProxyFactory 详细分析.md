# Spring ProxyFactory 详细分析

　　AOP 中 `ProxyFactory` 的子类有 `ProxyCreatorSupport`、`AdvisedSupport`、`ProxyConfig`。其中核心是 `ProxyCreatorSupport`，此类主要初始化了具体动态代理方案。其他 `AdvisedSupport`、`ProxyConfig` 主要是围绕 AOP 相关配置进行封装。

> ProxyFactory 基本涵盖了 Spring AOP 的基本实现。了解完成 ProxyFactory 后可以进入了解 ProxyFactoryBean，ProxyFactoryBean 进而结合 FactoryBean[1] 功能可以方便使用已经注册在容器内部的实现类。
>

## 1. 初始化 ProxyFactory

　　初始化时，ProxyCreatorSupport 默认构造函数初始化了一个默认的 AopProxyFactory [2]，此 AopProxyFactory 主要是依据不同的条件下面去使用 JDK 还是 CGLib 的动态代理。

```java
public class ProxyCreatorSupport extends AdvisedSupport {
	private AopProxyFactory aopProxyFactory;
...
	/**
	 * Create a new ProxyCreatorSupport instance.
	 */
	public ProxyCreatorSupport() {
		this.aopProxyFactory = new DefaultAopProxyFactory();
	}
...
}
```

## 2. 设置代理相关配置

```java
//这目标对象
ProxyFactory.setTarget(Object target);

//设置切面
ProxyFactory.addAdvice(Advice advice);
```

## 3. 获取代理对象 ProxyFactory#getProxy()

　　ProxyFactory#getProxy() 其实就是调用了 ProxyFactory#createAopProxy()，这里就是调用DefaultAopProxyFactory#createAopProxy(AdvisedSupport config) ，设置代理相关参数和返回AopProxy。

```java
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {

	@Override
	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
			return new ObjenesisCglibAopProxy(config);
		}
		else {
			return new JdkDynamicAopProxy(config);
		}
	}

```

　　再调用 AopProxy#getProxy() 方法返回真正的对象。到这里，表面那一层的调用说明这里即可。

## 4. AopProxy

### 4.1 JdkDynamicAopProxy

　　使用 JdkDynamicAopProxy 动态代理的话，只能代理接口类。JDK的动态代理主要是实现 InvocationHandler 接口。

```
public interface InvocationHandler {
    //调用接口里面的方法时，就会调用以下方法
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

　　也就是说核心在 JdkDynamicAopProxy#invoke(Object proxy, Method method, Object[] args) 方法。而判断那些方法需要增强的判断在

```java
List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
```

　　这里巧妙利用拦截器的个数来控制是否需要调用 Advice。这也就是为什么JDK的动态代理运行比较慢，是需要在 invoke 里面进行过滤执行。

> 假如需要自定义判断处理逻辑自己实现 AdvisorChainFactory 后，调用 `AdvisedSupport#setAdvisorChainFactory(AdvisorChainFactory advisorChainFactory)` 设置。
>

　　<!--具体的接口方法是一定会调用的，只是前面需要不需要有拦截器，很巧妙的运用的方法拦截器 （MethodInterceptors），如果PointCut 通过的话，就调用 MethodInterceptors 的链，否则就直接调用。

　　需要具体研究出如何实现的，实现的设计模式是怎么样的-->

## 5. AOP 基础

### 5.1 Advice 通知

　　具体做事情的类，也就是说你需要如何做或者做什么，都在 Advice 里面实现，目前常用的有

* AfterReturningAdvice：方法执行后运行
* MethodBeforeAdvice：方法调用前执行
* ThrowsAdvice：出现异常后调用。

### 5.2 Pointcut

　　定义什么样的东西需要被拦截。比如说以 get 开头我就拦截。常用有：

* NameMatchMethodPointcut：通过名称
* AspectJExpressionPointcut：通过使用一套表达式

### 5.3 Advisor

　　Advisor = Advice + Pointcut。常用有：

* AspectJExpressionPointcutAdvisor
* NameMatchMethodPointcutAdvisor

## 6. 总结

　　在AOP构建主线的过程中，提供了很多扩展的空间。比如：

* AopProxyFactory：如果不满足现在有的 JDK和CGLib 的动态代理实现，可以自定义实现。
* AdvisedSupportListener：可以注册监听动作
* AdvisorChainFactory：在具体实现调用增强的时候可以进行优化

　　使用的注解有：

```java
//标记为切面类
@Aspect
//拦截点
@Pointcut
//前置
@Before
//后置
@After
//环绕
@Around
```

## 7. 注释

　　[1]：支持使用已经注册到IoC容器的相关类，包括配置，切面等。

　　[2]：默认实现 DefaultAopProxyFactory，如果有其他的动态代理实现方式，可以去实现 接口 `AopProxyFactory`，然后使用 `ProxyCreatorSupport#setAopProxyFactory(AopProxyFactory aopProxyFactory)`。
