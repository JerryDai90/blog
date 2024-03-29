# Spring xml 解析扩展

　　Spring 配置文件 xml 是可以通过注册命名空间来达到解析扩展的。也就是说 AOP、TX、等都是通过扩展命名空间来解析数据的。定义自己的命名解析需要有几个步骤，主要采用[策略模式](https://www.runoob.com/design-pattern/strategy-pattern.html)进行开发，Github 实例代码：[https://github.com/JerryDai90/java-case/tree/master/spring/xml-extension](https://github.com/JerryDai90/java-case/tree/master/spring/xml-extension)

#### 1. 自定义 DefinitionParser

　　需要定义解析 xml `DefinitionParser`，如 `ConfigBeanDefinitionParser.java`

#### 2、定义 Handler

　　定义 `Handler`，此类用于注册到 `Spring` 处理类池里面，里面需要在 init 方法中注册实现类。

> 比如 AOP 就定义了 `config`、`aspectj-autoproxy`、`scoped-proxy`、`spring-configured` 这些元素就使用
>

```java
public class AopNamespaceHandler extends NamespaceHandlerSupport {
	@Override
	public void init() {
		// In 2.0 XSD as well as in 2.1 XSD.
		registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
		registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
		registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());

		// Only in 2.0 XSD: moved to context namespace as of 2.1
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
	}
}
```

#### 3. 定义 xml 命名空间

　　比如 AOP 的命名空间 `xmlns:aop="http://www.springframework.org/schema/aop"`

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

```

　　与 DSD 文件定义

#### 4、添加 spring.handlers、spring.schemas 文件

　　需要在 MATE-INF 中 增加 spring.handlers 文件。命名空间需要指定 Handler 类。

```
http\://www.springframework.org/schema/aop=org.springframework.aop.config.AopNamespaceHandler
```

　　需要在 MATE-INF 中 增加 spring.schemas 文件，对应命名空间对应的 xsd 文件。

#### 5. 关键类

　　DefaultNamespaceHandlerResolver：解析加载到 ClassLoader 里面的  spring.handlers 文件。
NamespaceHandlerSupport：注册标签对应的解析类

#### 6. 小结

　　这样就可完成扩展 Spring xml 的解析并且实现相应的业务。大概划的处理图。标准的配置流程可以参考 spring-aop-5.2.3.RELEASE.jar 配置。  
![w400](http://img.lsof.fun/2020-05-04-15885707470215.jpg)

* spring 初始化的时候就会去 ClassLoader 找到所有的 META-INF/spring.handlers 文件。
* 加载文件中的 Handler，调用 init 进行初始化（这里初始化就直接映射了命名空间与 element 之间的关系）。
* 真正解析的的时候拿到当前的 element 后，即可后去其命名空间，通过命名空间找到相应的 parser 对象进行解析。

#### 7. 其他

　　debug 看代码的时候，断点 DefaultNamespaceHandlerResolver#getHandlerMappings() 方法里面的时候，handlerMappings 成员变量都是有值的。以为有什么黑魔法，百思不得解。最后发现另外以为大神也遇到，附上链接：https://www.cnblogs.com/developer_chan/p/10002492.html
