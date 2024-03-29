服务使用之间如果使用 `feign` 相互调用的话，无论是 `POST` 或 `GET` 请求，如果携带的数据过长的话，会导致丢失部分数据或者报错。解决方法很简单。就是加大服务提供者的限制，如下：
修改 `yml` 或 `properties` 配置文件：

```yml
server:
    port: 4450
    # 增加请求头接受大小
    max-http-header-size: 10485760
```

## 1. 问题排查过程

### 1.1 现象

　　调用说明：A 服务调用 B 服务，前端提交到 A 服务的时候数据比较少，经过 A 加工后（数据多），调用 B 进行处理。

1. 服务 A 调用服务 B 的时候，服务 A 抛出异常，状态码是 `400`，而 B 服务没有异常。
2. 控制台信息
   ![w400](http://img.lsof.fun/2020-03-08-15836550983782.jpg)

### 1.2 问题排查

1. 一开始以为是发送方做了数据控制，截断了数据的发送。于是乎觉得是 `fegin` 的 `HttpClient` 的问题？使用 `Charles` 调试数据请求，发现不是。这样就断定了是服务提供者的问题。
2. 后面想起 `spring boot` 也是使用 `tomcat` 容器部署的，`tomcat` 本身也存在数据请求的限制，找到了 `POST` 相关配置如下：

```properties
# tomcat 配置
server.tomcat.max-http-form-post-size
# jetty 配置
server.jetty.max-http-form-post-size
```

　　还是解决不了。

## 2. 疑惑

### 2.1 为什么是是配置 max-http-header-size 就可以了。难道是 `fegin` 是把请求放到头部？需要需要重新分析请求的数据？

　　官方对此参数的说明

| Key	Value                   | Default | Description                              |
| ----------------------------- | --------- | ------------------------------------------ |
| server.max-http-header-size | 8KB     | Maximum size of the HTTP message header. |

> 摘抄于 https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#server-properties
>

　　依据说明寻找 `HTTP message header` 规范

> 根据不同上下文，可将消息头分为：
>
> * General headers: 同时适用于请求和响应消息，但与最终消息主体中传输的数据无关的消息头。
> * Request headers: 包含更多有关要获取的资源或客户端本身信息的消息头。
> * Response headers: 包含有关响应的补充信息，如其位置或服务器本身（名称和版本等）的消息头。
> * Entity headers: 包含有关实体主体的更多信息，比如主体长(Content-Length)度或其 MIME 类型。
>
> 摘抄于 [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)
>

　　并未找到相关头部具体的定义消息，但经过实际的代码试验，`URL` 就是头部的一部分，默认是 `8KB`。超过后就会拒绝。（在头部设置超过 `8KB` 的数据也是同样的异常）。

　　控制台报错的错误

### 2.2 找出 fegin 拼接到 url 上的原因

　　通过 debug 找到关键方法：

```java
//解析好了fegin 配置，准备调用远程方法
SynchronousMethodHandler#invoke(Object[] argv);

//执行远程调用和编码，
SynchronousMethodHandler#executeAndDecode(RequestTemplate template);

//这里转换成 Request 对象的时候，已经把参数拼接到 URL 上了，也就是说 POST 变成了GET 请求
SynchronousMethodHandler#targetRequest(RequestTemplate template)
```

　　也就是说，如果要彻底解决问题，需要更换底层相关实现。另外如果可以修改源代码，解决也很简单，只需要依据请求的方法来构建是否把数据放到 body 还是 url 上面即可。

### 2.3 使用 httpClient 代替默认实现

　　增加 `maven` 依赖:

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.11</version>
</dependency>

<!-- https://mvnrepository.com/artifact/io.github.openfeign/feign-httpclient -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>10.8</version>
</dependency>
```

　　在 yml 文件中增加配置:

```yml
feign:
  httpclient:
    enabled: true
```

　　增加后启动依旧 `POST` 变 `GET` 请求。

　　**以下是原因分析：**

　　先回经过 `ApacheHttpClient#toHttpUriRequest(Request request, Request.Options options);` 方法，由于前面构建的 `Request` 对象就没有 `body` 信息。所以默认赋予一个空的 Entity。如下图：

　　![w500](http://img.lsof.fun/2020-03-08-15836757341084.jpg)

　　其次调用了 `RequestBuilder#build();` 构建的时候，下图 ② 处又将重新参数赋予到 url 上了，又重复出现一样的问题。如下图：  
![w500](http://img.lsof.fun/2020-03-08-15836755685690.jpg)

#### 2.3.1 修改 feign 源码

　　上面我们已经确认了问题所在，这个时候只需要修改部分源码即可解决这个问题。复制 feign.httpclient.ApacheHttpClient 到你工程路径下（注意包名也要一致），然后注释掉下面这句。

　　![w600](http://img.lsof.fun/2020-03-09-15837611769939.jpg)

　　去掉以下依赖

```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>10.8</version>
</dependency>
```

　　重新导入刷新工程即可。这个时候就是正确的 `GET` 和 `POST`

## 3. 参考文献

　　[https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#server-properties](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#server-properties)

　　[https://www.jianshu.com/p/11710629c226](https://www.jianshu.com/p/11710629c226)

　　**HTTP Body 相关规范**

　　[https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Messages
](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Messages)[https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST)

　　
