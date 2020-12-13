# Spring Security 认证
> 下来总结结合官网教程理解整理，如果有出现逻辑有误，请指出。

## 1. 总结
Security 认证都是基于围绕 Filter来展开，比如默认的 `/login` 接口采用的是 `UsernamePasswordAuthenticationFilter` 过滤器进行处理（访问路径进行匹配 RequestMatcher ）。而注入到过滤器链入口是 `HttpSecurity`。

> HttpSecurity 相当于我们的XML文件。只是这里采用的是编码的方式进行声明。

## 2. 核心类说明
以下摘抄于翻译：

**架构组件**

* SecurityContextHolder - Spring Security 在其中存储 SecurityContextHolder,用于存储通过身份验证的人员的详细信息.
* SecurityContext - 从 SecurityContextHolder 获得,并包含当前经过身份验证的用户的 Authentication .
* Authentication - 可以是 AuthenticationManager 的输入,用户提供的用于身份验证的凭据或来自 SecurityContext 的当前用户.
* GrantedAuthority - 在 Authentication 授予委托人的权限 (即角色,作用域等) .
* AuthenticationManager - 定义 Spring Security 的过滤器如何执行身份验证的API.（其实就是认证，可以直接在这里写逻辑具体认证逻辑）
* ProviderManager - AuthenticationManager 的最常见实现.（实现了可以通过 AuthenticationProvider#supports()方法来判断使用什么验证方式 ）
* AuthenticationProvider - 由 ProviderManager 用于执行特定类型的身份验证.（具体的的认证实现，由AuthenticationManager 持有）
* 使用 AuthenticationEntryPoint 请求凭据 -带 AuthenticationEntryPoint 的请求凭据-用于从客户端请求凭据 (即重定向到登录页面,发送 WWW-Authenticate 响应等) .
* AbstractAuthenticationProcessingFilter - 用于验证的基本过滤器. 这也为高级的身份验证流程以及各个部分如何协同工作提供了一个好主意.

这几个类非常重要，验证的过程基本都使用到这些类

![](http://img.lsof.fun/2020-12-13-16074401507745.jpg)

做重看 `HttpSecurityConfiguration#httpSecurity()` 方法，这里初始化了`HttpSecurity` 对象AuthenticationManagerBuilder 的默认装配实现 DefaultPasswordEncoderAuthenticationManagerBuilder

## 3. 整体的路程

### 3.1 认证登陆逻辑分析

WebSecurityConfigurerAdapter 自动装配了路径为 `/login` 这个路径为登陆认证请求路径。如下：

```java
protected void configure(HttpSecurity http) throws Exception {
	this.logger.debug("Using default configure(HttpSecurity). "
			+ "If subclassed this will potentially override subclass configure(HttpSecurity).");
	http.authorizeRequests((requests) -> requests.anyRequest().authenticated());
	http.formLogin();
	http.httpBasic();
}
```

着重看  `http.formLogin();`

```java
public FormLoginConfigurer<HttpSecurity> formLogin() throws Exception {
    //这里关键
    return getOrApply(new FormLoginConfigurer<>());
}
```
这里构建了 `FormLoginConfigurer` 相关逻辑，且定义了登陆需要的字段信息。关键是实例化了UsernamePasswordAuthenticationFilter (AbstractAuthenticationProcessingFilter)，这个类继承了Filter 。主要的使用 `attemptAuthentication()` 方法来拦截登陆请求。此方法是由下面方法调起。

```java
AbstractAuthenticationProcessingFilter.doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
```

回调方法 `attemptAuthentication()` 逻辑方法运行的类逻辑图。 
 ![w500](http://img.lsof.fun/2020-12-13-16078545172829.jpg)

接口类逻辑图说明：
1、 ProviderManager（AuthenticationManager） 持有所有的 AuthenticationProvider 对象（注入此对象请查看 `AuthenticationProvider 使用说明`）。

> UsernamePasswordAuthenticationFilter 构建用户输入的对象时已经决定用户可以使用的 AuthenticationProvider。
![](http://img.lsof.fun/2020-12-13-16078635109165.jpg)

2、通过AuthenticationProvider.supports()方法匹配合适的 AuthenticationProvider 来返回 Authentication （里面囊括用户、认证信息）
3、如果是类型是 DaoAuthenticationProvider 的话，会使用 PasswordEncoder、UserDetailsService 来辅助认证。

> 思考：为什么要设计的那么复杂？

## 4. 实战
### 4.1 自定义一个认证
需求：使用手机+短信密码进行登录。下面忽略发送短信动作，只做认证

**1、创建数据传递类**: 用于包装用户传输的信息

```java
public class SMSAuthenticationToken extends AbstractAuthenticationToken {

    private Object principal;//手机号
    private Object credentials;//验证码

    public SMSAuthenticationToken(Object principal, Object credentials) {
        super(null);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(false);
    }
    public SMSAuthenticationToken(Object principal, Object credentials, Collection<? extends GrantedAuthority> authorities) {
        super(null);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(false);
    }

    @Override
    public Object getCredentials() {
        return this.credentials;
    }

    @Override
    public Object getPrincipal() {
        return this.principal;
    }

}
```

**2、创建创建过滤器类**
增加我们需要的路径过滤器（告诉容器我们要拦截那些路径来出来），访问路径为 /smsLogin

```java
public class SMSAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    public SMSAuthenticationFilter(AuthenticationManager authenticationManager) {
        super(new AntPathRequestMatcher("/smsLogin", "GET"));
        setAuthenticationManager(authenticationManager);
    }
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {

        String phone = request.getParameter("phone");
        String code = request.getParameter("code");
        SMSAuthenticationToken token = new SMSAuthenticationToken(phone, code);
        return getAuthenticationManager().authenticate(token);
    }
}
```
**3、创建具体的验证器类**

```java
public class SMSAuthenticationProvider implements AuthenticationProvider {
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {

        SMSAuthenticationToken token = (SMSAuthenticationToken) authentication;
        //验证code
        if(!"1111".equals(token.getCredentials())){
            throw new BadCredentialsException("验证码不对");
        }
        //TODO 构建权限，获取权限
        ArrayList<GrantedAuthority> grantedAuthorities = GrantedAuthority();
        SMSAuthenticationToken newToken = new SMSAuthenticationToken(token.getPrincipal(), token.getCredentials(), grantedAuthorities);

        return newToken;
    }
    private ArrayList<GrantedAuthority> GrantedAuthority() {
        ArrayList<GrantedAuthority> grantedAuthorities = new ArrayList<>();
        grantedAuthorities.add(new SimpleGrantedAuthority("ROLE_USER"));

        return grantedAuthorities;
    }
    @Override
    public boolean supports(Class<?> authentication) {
        return SMSAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

**4、声明注入到过滤器中**

```java
@Configuration
public class TestWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {

    //由于5.x 版本是无法获取到 AuthenticationManager，需要声明
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Autowired
    AuthenticationManager authenticationManager;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authenticationProvider(new SMSAuthenticationProvider())
                .addFilterAt(new SMSAuthenticationFilter(authenticationManager), UsernamePasswordAuthenticationFilter.class);
    }
}
```

## 5. 其他说明
### 5.1 AuthenticationProvider 使用说明

SpringBoot 默认使用 DaoAuthenticationProvider。此提供者可以使用 UserDetailsService 、PasswordEncoder 来扩展使用。
扩展性说明

* 但是可以通过 HttpSecurity.authenticationProvider() 增加 AuthenticationProvider。
* 如果 AuthenticationProvider.supports 都支持，则按顺序调用，其中有一个调用返回非空且不抛异常则不再往下执行直接返回。（这种适用于多重系统认证，按 A->B->C 顺序执行尝试认证 ）


