# Spring ProxyFactory 详细分析

AOP 中 `ProxyFactory` 的子类有 `ProxyCreatorSupport`、`AdvisedSupport`、`ProxyConfig`。其中核心是 `ProxyCreatorSupport`，此类主要初始化了具体动态代理方案。其他 `AdvisedSupport`、`ProxyConfig` 主要是围绕 AOP 相关配置进行封装。

> ProxyFactory 基本涵盖了 Spring AOP 的基本实现。了解完成 ProxyFactory 后可以进入了解 ProxyFactoryBean，ProxyFactoryBean 进而结合 FactoryBean[1] 功能可以方便使用已经注册在容器内部的实现类。

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

再调用 AopProxy#getProxy() 方法放回真正的对象。

## 注释

[1]：支持使用已经注册到IoC容器的相关类，包括配置，切面等。

[2]：默认实现 DefaultAopProxyFactory，如果有其他的动态代理实现方式，可以去实现 接口 `AopProxyFactory`，然后使用 `ProxyCreatorSupport#setAopProxyFactory(AopProxyFactory aopProxyFactory)`。



