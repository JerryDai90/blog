# 使用 feign 调用服务时，Post 变 Get 请求的解决方案

## 1. 问题

使用的是 `2.1.1` 版本的 `feign`，进过大量的测试，无论是标准是 `@PostMapping` 还是 `@GetMapping`，只要参数标注 `@RequestParam`，调用的时候就一律都用 `Get` 请求，也就是说把参数拼接到 `URL` 上。如果想使用Post 请求，需要在参数标记 `@RequestBody`，这样无论是 `Get` 还是 `Post` 都一律使用 `Post`。

以下有几种的实现方式可以做到配置区分 `Post` 和 `Get` 请求。下列方案可能有缺陷，请慎用。

## 2. 解决办法
### 2.1 增加 feign 过滤器

增加的 feign 拦截器，此拦截器是在 RequestTemplate 构建完成后执行的，我们手动再进行加工一下。把Post 相关的参数写入到 Body 里面，从而达到目的。

**1、增加feign的拦截器 FeignRequestInterceptor**

```java
import com.xuanyuan.common.utils.StringUtils;
import feign.*;
import feign.template.QueryTemplate;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.collections.MapUtils;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;extHolder;
import org.springframework.web.context.request.ServletRequ
import org.springframework.web.context.request.RequestContestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.*;

import static org.springframework.http.MediaType.APPLICATION_FORM_URLENCODED_VALUE;

@Slf4j
@Component
public class FeignRequestInterceptor implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate template) {

        /处理POST请求的时候转换成了GET的bug
        if (HttpMethod.POST.name().equals(template.method()) && template.requestBody().length() == 0 && !template.queries().isEmpty()) {
            Object object = MapUtils.getObject(template.headers(), HttpHeaders.CONTENT_TYPE);
            if (null != object) {

                if (((Collection) object).contains(APPLICATION_FORM_URLENCODED_VALUE)) {

                    StringBuilder builder = new StringBuilder();
                    Map<String, Collection<String>> queries = template.queries();
                    Iterator<String> queriesIterator = queries.keySet().iterator();

                    while (queriesIterator.hasNext()) {
                        String field = queriesIterator.next();
                        Collection<String> strings = queries.get(field);
                        //由于参数已经做了url编码处理，这里直接拼接即可
                        builder.append(field + "=" + StringUtils.join(strings, ","));
                        builder.append("&");
                    }

                    template.body(Request.Body.encoded(builder.toString().getBytes(), template.requestCharset()));
                    template.queries(null);
                }
            }
        }
        log.debug("FeignRequestInterceptor:{}", template.toString());
    }
}
```

**2、feign 接口定义**

```java
// consumes = "application/x-www-form-urlencoded" 是必须设置的，否则不会进入上面写的处理过程
@PostMapping(value = "/youUrl", consumes = "application/x-www-form-urlencoded")
ResultBody<Map<String,Object>> youMethod(@RequestParam("a") String a, @RequestParam("b") String b);
```

### 2.2 使用 httpClient 代替默认实现

使用 httpClient 的实现，虽然 httpClient 里面的处理是会把URL里面的参数挪动到Body里面，但是由于ApacheHttpClient 的实现上和后面的处理有冲突。详细分析请看另外一篇分析 [《Spring boot 使用 feign 调用参数过长（Post变Get）》](https://lsof.fun/15834600207814.html)，但是需要修改 feign 的源码，这里直接采用在项目覆盖默认的实现。从而达到目的。

**1、增加 `maven` 依赖:**

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.11</version>
</dependency>

<!--<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>10.8</version>
</dependency>-->
```

**2、在yml文件中增加配置**

```yml
feign:
  httpclient:
    enabled: true
```

> 这个是 FeignAutoConfiguration 自动配置

在根目录增加 `feign.httpclient` package，把 feign-httpclient 中的 ApacheHttpClient 直接复制到此目录，然后注释掉此句话即可。

![w500](http://img.lsof.fun/2020-03-09-15837611769939.jpg)

代码详细实例可以查看 [https://github.com/JerryDai90/sping-boot-experiment/blob/master/feign/src/main/java/fun/lsof/feign/fix/urlencode/FeignRequestInterceptor.java](https://github.com/JerryDai90/sping-boot-experiment/blob/master/feign/src/main/java/fun/lsof/feign/fix/urlencode/FeignRequestInterceptor.java)

## 3. 思考
这几天看了比较多的源码，feign 相关源码不应该犯这种错误，把 POST 参数放到 URL 上。这个原因我的猜想是由于无论是放到 body 还是 header 上面，其实效果都是一样的（放到 url和在body编码方式和组合都是一样的），header 的默认大小限制就小一点（8K或者更少），而 Post 则大一点（2M+）。换句话来说，其实可以忽略这个问题，直接在配置文件中加大限制即可

```yml
server:
    port: 4450
    # 增加请求头接受大小
    max-http-header-size: 10485760
```

