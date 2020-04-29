# JDK Proxy 代理源码分析

过程说明：动态生成目标接口的 Class 代理类，这个代理类是实现了接口中的所有方法。然后再把此class加载到内存中。调用代理类方法的时候代理类去调用实际对象方法。

## 1. 分析生产的过程
Proxy#newProxyInstance 中的代码就描述上面说的过程，
1. 生成代理类 class 对象
2. 构建代理类实例

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)  throws IllegalArgumentException  {
    ...
    //依据接口生产代理对象的 class 对象
    Class<?> cl = getProxyClass0(loader, intfs);
    ...    
    //生成代理对象实例
    final Constructor<?> cons = cl.getConstructor(constructorParams);
    final InvocationHandler ih = h;
    if (!Modifier.isPublic(cl.getModifiers())) {
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                cons.setAccessible(true);
                return null;
            }
        });
    }
    return cons.newInstance(new Object[]{h});
    ...
}

```

其中核心的代码生成 `getProxyClass0(loader, intfs)`，此方法里面只需要关注 `proxyClassCache`（WeakCache类） 这个成员变量。此类是用于缓存代理对象的。

```
/**
 * a cache of proxy classes
 */
private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
    proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
```
> 构建 proxyClassCache 有2个参数，其中关键的是 ProxyClassFactory ①

`WeakCache#get` 方法是核心方法，这里是构建并且缓存对象。

```java
public V get(K key, P parameter) {

    ...
    //关键
    while (true) {
        if (supplier != null) {
            // supplier might be a Factory or a CacheValue<V> instance
            // 这里已经说明白了，第一次的话都是 Factory 提供的实例
            V value = supplier.get();
            if (value != null) {
                return value;
            }
        }
        // else no supplier in cache
        // or a supplier that returned null (could be a cleared CacheValue
        // or a Factory that wasn't successful in installing the CacheValue)

        // lazily construct a Factory
        if (factory == null) {
            //关键在此处，构建完成 Factory 后，我们关注一下 Factory 的实现
            factory = new Factory(key, parameter, subKey, valuesMap);
        }

        if (supplier == null) {
            supplier = valuesMap.putIfAbsent(subKey, factory);
            if (supplier == null) {
                // successfully installed Factory
                supplier = factory;
            }
            // else retry with winning supplier
        } else {
            if (valuesMap.replace(subKey, supplier, factory)) {
                // successfully replaced
                // cleared CacheEntry / unsuccessful Factory
                // with our Factory
                supplier = factory;
            } else {
                // retry with current supplier
                supplier = valuesMap.get(subKey);
            }
        }
    }
    ...
}

```

下面来到 `Factory#get` 方法中，其实核心就是 `valueFactory.apply(key, parameter)`，其实就是调用了 ProxyClassFactory ①

```java

private final class Factory implements Supplier<V> {

    @Override
    public synchronized V get() { 
        ...
        // serialize access
        // create new value    
        V value = null;
        try {
            //核心就是 valueFactory.apply(key, parameter)，其实就是调用了 ProxyClassFactory ①
            value = Objects.requireNonNull(valueFactory.apply(key, parameter));
        } finally {
            if (value == null) { // remove us on failure
                valuesMap.remove(subKey, this);
            }
        }
        ...
        return value;
    }
}
```

现在我们目光回到 `ProxyClassFactory#apply` 方法

```java
public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

    //这里就生产了代理类的字节码流
    byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);
    try {
        return defineClass0(loader, proxyName, proxyClassFile, 0, proxyClassFile.length);
    } catch (ClassFormatError e) {
        /*
         * A ClassFormatError here means that (barring bugs in the
         * proxy class generation code) there was some other
         * invalid aspect of the arguments supplied to the proxy
         * class creation (such as virtual machine limitations
         * exceeded).
         */
        throw new IllegalArgumentException(e.toString());
    }
}

```

也就是说，我们也可以手动去调用 `ProxyGenerator#generateProxyClass` 去生成我们的字节码。

### 1.1 小结

通过以上的分析，其实JDK动态代理核心就是 `ProxyGenerator#generateProxyClass`，更多的旁枝末节更多是辅助性的，比如缓存等。

## 2. 如何实现对目标方法的调用

我们回顾一下动态代理是如何使用的，

```java
//调用生成代理类，其中关键需要传入interfaces和InvocationHandler。
Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```

其中 `InvocationHandler` 我们会这样实现

```java
// 其中我们会实现这个方法，在这个方法里面调用实际的代理对象
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
    //前置逻辑
    Object obj = method.invoke(被代理的对象实例，args);
    //后置逻辑
    return obj;
}
```

#### 2.1 解析生成的代理类

了解完成后如何调用，我们看一下生成的 class 反编译看看。

我们定义一个接口 `ICar`

```java
public interface ICar {
    void start();
    void run();
    void stop();
}
```

通过动态代理生成的 class 

```java
import fun.lsof.javacore.proxy.dynamic.ICar;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class Proxy1 extends Proxy implements ICar {

    // 拥有具体对象的方法对象
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m5;
    private static Method m4;
    private static Method m0;

    // 默认构造函数需要传入 InvocationHandler 
    public Proxy1(InvocationHandler paramInvocationHandler) {
        super(paramInvocationHandler);
    }
    // 实现的方法
    public final void start() {
        try {
            this.h.invoke(this, m4, null);
            return;
        } catch (Error | RuntimeException localError) {
            throw localError;
        } catch (Throwable localThrowable) {
            throw new UndeclaredThrowableException(localThrowable);
        }
    }
    // 实现的方法
    public final void run() {
        try {
            this.h.invoke(this, m3, null);
            return;
        } catch (Error | RuntimeException localError) {
            throw localError;
        } catch (Throwable localThrowable) {
            throw new UndeclaredThrowableException(localThrowable);
        }
    }
    // 实现的方法
    public final void stop() {
        try {
            this.h.invoke(this, m5, null);
            return;
        } catch (Error | RuntimeException localError) {
            throw localError;
        } catch (Throwable localThrowable) {
            throw new UndeclaredThrowableException(localThrowable);
        }
    }
    
    // 初始化全局变量中的方法，通过反射拿到Method对象
    static {
        try {  
            m3 = Class.forName("fun.lsof.javacore.proxy.dynamic.ICar").getMethod("run", new Class[0]);
            m5 = Class.forName("fun.lsof.javacore.proxy.dynamic.ICar").getMethod("stop", new Class[0]);
            m4 = Class.forName("fun.lsof.javacore.proxy.dynamic.ICar").getMethod("start", new Class[0]);
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
            m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
            return;
        } catch (NoSuchMethodException localNoSuchMethodException) {
            throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
        } catch (ClassNotFoundException localClassNotFoundException) {
            throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
        }
    }
    
    public final boolean equals(Object paramObject) {
        ...
    }
    public final int hashCode() {
        ...
    }
    public final String toString() {
        ...
    }
}
```

看到以上自动生成的代码，`Proxy1` 初始化的时候，就会加载静态函数，其中静态代码块中是枚举了所有接口（ICar）中类的方法。拥有接口中的方法是为了反射可以调用。再看到 start、run、stop 方法，每个方法都回调了 InvocationHandler#invoke方法。从而达到对方法的前后增强

```java
// 提供具体代理的对象 proxy、代理方法的 method、代理方法的参数 args
invoke(Object proxy, Method method, Object[] args) 
```

> 不依靠Proxy来实现动态代理实现方法：[https://github.com/JerryDai90/java-case/tree/master/java-core/proxy/src/main/java/fun/lsof/javacore/proxy/dynamic](https://github.com/JerryDai90/java-case/tree/master/java-core/proxy/src/main/java/fun/lsof/javacore/proxy/dynamic)

## 3. 总结
其实 JDK 的动态代理就是动态生成字节码而已，然后再使用反射技术对接口中的方法进行引用。通过反射即可调用被代理对象的方法。从而达到代理的目的。

## 4. 扩展

高仿一个 ProxyFactory，Github：[https://github.com/JerryDai90/java-case/tree/master/java-core/proxy/src/main/java/fun/lsof/javacore/proxy/dynamic/aop](https://github.com/JerryDai90/java-case/tree/master/java-core/proxy/src/main/java/fun/lsof/javacore/proxy/dynamic/aop)

