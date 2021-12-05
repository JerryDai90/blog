# ProxyFactory 之 CGLib 代理分析

## 1. Enhancer 的基本使用

　　原生直接使用 Enhancer 的话，测试代码如下

```java
public static void main(String[] args) {

    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(ArrayList.class);
    enhancer.setCallback(new MethodInterceptor() {
        @Override
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
            //这里就是每个方法都会经过的拦截器
            //通过在这里加工就可以做到前、后、异常等切面
            System.out.println("call method: " + method.getName());
            return methodProxy.invokeSuper(o, objects);
        }
    });
    ArrayList o = (ArrayList) enhancer.create();
    o.clear();
}
```

> 基础使用的话基本和JDK 的那个 `InvocationHandler` 基本无差别。
>

## 2. CglibAopProxy 分析

　　CglibAopProxy 是整个 AOP 使用 Cglib 功能类。使用过程不解释了。可以把程序跑起来，然后看看如何执行的。debug 物料 [SpringAopHardCodedByCGlibTest.java](https://github.com/JerryDai90/java-case/blob/master/spring/aop/src/main/java/fun/lsof/spring/aop/dynamicproxy/cglib/SpringAopHardCodedByCGlibTest.java)

　　其实整个动态代理的关键就是回调，也就是 `Enhancer#setCallback` 函数。我们做重看一下 CglibAopProxy#getCallbacks 方法，下面方法已经精简出来，核心需要关注的。

```java
	private Callback[] getCallbacks(Class<?> rootClass) throws Exception {
		// Parameters used for optimization choices...
		boolean exposeProxy = this.advised.isExposeProxy();
		boolean isFrozen = this.advised.isFrozen();
		boolean isStatic = this.advised.getTargetSource().isStatic();

		// Choose an "aop" interceptor (used for AOP calls).
		Callback aopInterceptor = new DynamicAdvisedInterceptor(this.advised);
    ...
		
		Callback[] mainCallbacks = new Callback[] {
				aopInterceptor,  // for normal advice
				targetInterceptor,  // invoke target without considering advice, if optimized
				new SerializableNoOp(),  // no override for methods mapped to this
				targetDispatcher, this.advisedDispatcher,
				new EqualsInterceptor(this.advised),
				new HashCodeInterceptor(this.advised)
		};
		Callback[] callbacks;
...
			// Now copy both the callbacks from mainCallbacks
			// and fixedCallbacks into the callbacks array.
			callbacks = new Callback[mainCallbacks.length + fixedCallbacks.length];
...
		}
		else {
			callbacks = mainCallbacks;
		}
		return callbacks;
	}

```

　　我们看到，整个return 的 Callback[]里面 DynamicAdvisedInterceptor 是排在第一位的，我们着重关注一下。DynamicAdvisedInterceptor 也是一个 MethodInterceptor，也就是说方法调用的时候就会拦截到。就类是于我们自己写的callback函数的逻辑。通过这里面的逻辑进行前置后缀等逻辑。

# 3. DynamicAdvisedInterceptor 分析

　　这个类就是对前面定义的切面进行调用

```java
public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
	Object oldProxy = null;
	boolean setProxyContext = false;
	Object target = null;
	TargetSource targetSource = this.advised.getTargetSource();
	try {
		if (this.advised.exposeProxy) {
			// Make invocation available if necessary.
			oldProxy = AopContext.setCurrentProxy(proxy);
			setProxyContext = true;
		}
		// Get as late as possible to minimize the time we "own" the target, in case it comes from a pool...
		target = targetSource.getTarget();
		Class<?> targetClass = (target != null ? target.getClass() : null);
		//当前方法是否满足切面调用的定义（以什么开头等）
		List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
		Object retVal;
		// Check whether we only have one InvokerInterceptor: that is,
		// no real advice, but just reflective invocation of the target.
		if (chain.isEmpty() && Modifier.isPublic(method.getModifiers())) {
			// We can skip creating a MethodInvocation: just invoke the target directly.
			// Note that the final invoker must be an InvokerInterceptor, so we know
			// it does nothing but a reflective operation on the target, and no hot
			// swapping or fancy proxying.
			Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
			retVal = methodProxy.invoke(target, argsToUse);
		}
		else {
			// We need to create a method invocation...
			retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
		}
		retVal = processReturnType(proxy, target, method, retVal);
		return retVal;
	}
	finally {
		if (target != null && !targetSource.isStatic()) {
			targetSource.releaseTarget(target);
		}
		if (setProxyContext) {
			// Restore old proxy.
			AopContext.setCurrentProxy(oldProxy);
		}
	}
}
```

# 4. 总结

　　其实 CGLib 和 JDK Proxy 使用上大同小异，都是通过“方法的过滤器”来增强函数的功能。只是Spring 在封装的时候增加了亿点点细节。让我们可以扩展更加方便。
