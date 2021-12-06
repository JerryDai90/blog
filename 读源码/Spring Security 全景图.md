# Spring Security 全景图

　　下来总结结合官网教程整理。

## 1. 全局思想

　　整个过程都是基于 FilterChain、Filter 实现的，这张图非常重要如下图。

　　![](http://img.lsof.fun/2020-12-06-16071546837278.jpg)

　　其实这里的 Filter 就像平台 web.xml 中配置过滤器。由上图可知，Filter --> DispatcherServlet（Servlet） --> Controller。所以如果需要做拦截功能，可以在 Filter 中实现全局拦截处理。

### 1.1 DelegatingFilterProxy

　　![](http://img.lsof.fun/2020-12-06-16071549249548.jpg)  
它允许延迟查找 Filter bean 实例，换句话来说可以通过注册到 Spring 容器中的 Filter bean 是会加入到请求链中的去。如图一的 Filter。

### 1.2 FilterChainProxy

　　![](http://img.lsof.fun/2020-12-06-16071552874725.jpg)

　　FilterChainProxy 为真正执行相关动作类，通过不同的**路径**来匹配不同的 SecurityFilterChain

　　另外一个关键点是在 DelegatingFilterProxy 实际代理的对象是 FilterChainProxy。看 WebSecurityConfiguration 类中的 bean 实例化。

　　![](http://img.lsof.fun/2020-12-06-16071554772293.jpg)

　　WebSecurityConfiguration 的装配是由于注解 `@EnableWebSecurity` 生效了。

> 详情请看章节 “DelegatingFilterProxy 是如何融合 FilterChainProxy”
>

### 1.3 SecurityFilterChain

　　![](http://img.lsof.fun/2020-12-06-16071583014722.jpg)

　　![](http://img.lsof.fun/2020-12-06-16071583171990.jpg)

　　其实 FilterChainProxy 也是一个 Filter，FilterChainProxy 中持有 SecurityFilterChain list，对于不同的匹配路径规则走不同的 Filter。就如图五中的/api/**路径就走 SecurityFilterChain 0。 因为 FilterChainProxy 是 Filter，请求过来时 中

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain);
private void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain);
//关键这个方法，对应路径的过滤器
private List<Filter> getFilters(HttpServletRequest request);
```

## 2. 关键源码分析

### 2.1. DelegatingFilterProxy 是如何融合 FilterChainProxy

　　springBoot 中已经引入自动配置（其中包含了 Security 的自动配置类）
![](http://img.lsof.fun/2020-12-06-16063068582925.jpg)

　　重点看 `@SecurityAutoConfiguration` 注解

```java
@Import({ SpringBootWebSecurityConfiguration.class, WebSecurityEnablerConfiguration.class,
		SecurityDataConfiguration.class })
```

　　其中 `WebSecurityEnablerConfiguration` 中配置了 `@EnableWebSecurity`，

　　在看 `EnableWebSecurity` 如下，关键就是这些 Configuration 进行初始化。

```
@Import({ WebSecurityConfiguration.class, SpringWebMvcImportSelector.class, OAuth2ImportSelector.class,
		HttpSecurityConfiguration.class })
@EnableGlobalAuthentication
@Configuration
public @interface EnableWebSecurity {
...
}
```

　　DelegatingFilterProxy 会让 DelegatingFilterProxyRegistrationBean.getFilter() 进行初始化。
初始化的时候设置了一个 targetBeanName（此 bean 值为代理的 bean）

　　由于 DelegatingFilterProxy 也是一个过滤器。是由 servlet 容器进行初始化。请求先进过且进行 DelegatingFilterProxy.doFilter() 。doFilter 初始化代理 bean 的请求。然后调用实际的被代理对象的实际 doFilter 逻辑（实际代理对象也是一个 Filter）

　　DelegatingFilterProxy 中  
![](http://img.lsof.fun/2020-12-06-16063587494389.jpg)

　　而 `@Bean(name=springSecurityFilterChain)` 在 `WebSecurityConfiguration` 中定义了。

> 这里使用了委托代理模式
>

　　
