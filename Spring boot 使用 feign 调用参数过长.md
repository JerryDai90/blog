# Spring boot 使用 feign 调用参数过长

服务使用之间如果使用 `feign` 相互调用的话，无论是 `POST` 或 `GET` 请求，如果携带的数据过长的话，会导致丢失部分数据或者报错。解决方法很简单。就是加大服务提供者的限制，如下：
修改 `yml` 或 `properties` 配置文件：

```yml
server:
    port: 4450
    # 增加请求头接受大小
    max-http-header-size: 10485760
```

## 问题排查过程
1. 一开始以为是发送方做了数据控制，截断了数据的发送。于是乎觉得是 `fegin` 的`HttpClient` 的问题？使用`Charles` 调试数据请求，发现不是。这样就断定了是服务提供者的问题。

2. 后面想起 `spring boot` 也是使用 `tomcat`容器部署的，`tomcat` 本身也存在数据请求的限制，如下：

```properties
# tomcat 配置
server.tomcat.max-http-form-post-size
# jetty 配置
server.jetty.max-http-form-post-size
```
还是解决不了。突然看到了帖子，说配置头部即可解决问题。果然。

> 为什么是是配置 max-http-header-size 就可以了。难道是 `fegin` 是把请求放到头部？需要需要重新分析请求的数据？

<font color="red">待续。。。</font>

## 思考
**问：同样的数据，如果直接在使用前端的 Ajax 请求的话，会缺失数据吗？**
待验证。。。。


## 参考文献
https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#server-properties

