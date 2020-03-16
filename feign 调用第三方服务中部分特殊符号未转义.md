# feign 调用第三方服务中部分特殊符号未转义

开发过程中，发现+（加号）这个符号没有转义，导致再调用服务的时候把加号转义成空格了。导致后台获取到的数据会不正确。

## 1. 问题发现过程

feign 解析参数的时候，使用的标准是 [`RFC 3986`](https://www.ietf.org/rfc/rfc3986.txt)，这个标准的加号是不需要被转义的。其具体的实现是 `feign.template.UriUtils#encodeReserved(String value, String reserved, Charset charset)` 

## 2. 解决办法

**feign 调用过程**

```java
1. feign核心先将（定义好的feign接口）接口中的参数解析出来
2. 对接实际参数和接口参数（入参调用的参数）
3. 对入参的参数进行编码（UriUtils#encodeReserved）（问题出在这里）
4. 调用注册的 RequestInterceptor（自定义）
5. Encoder 实现类，这里是body里面的内容才会有调用（自定义）
6. 具体的http网络请求逻辑
```

依据上面的过程，我们可以实现一个 RequestInterceptor 拦截器，在这里对参数再次进行转义即可。

```
public void apply(RequestTemplate template) {

    Map<String, Collection<String>> _queries = template.queries();
    if (!_queries.isEmpty()) {
        //由于在最新的  RFC 3986  规范，+号是不需要编码的，因此spring 实现的是这个规范，这里就需要参数中进行编码先，兼容旧规范。
        Map<String, Collection<String>> encodeQueries = new HashMap<String, Collection<String>>(_queries.size());

        Iterator<String> iterator = _queries.keySet().iterator();
        Collection<String> encodeValues = null;
        while (iterator.hasNext()) {
            encodeValues = new ArrayList<>();

            String key = iterator.next();
            Collection<String> values = _queries.get(key);

            for (String _str : values) {
                _str = _str.replaceAll("\\+", "%2B");
                encodeValues.add(_str);
            }
            encodeQueries.put(key, encodeValues);
        }
        template.queries(null);
        template.queries(encodeQueries);
    }
}
```

上面是代码片段，详细请查看 [FeignRequestInterceptor.java](https://github.com/JerryDai90/sping-boot-experiment/blob/master/feign/src/main/java/fun/lsof/feign/fix/urlencode/FeignRequestInterceptor.java)

## 3. 疑问
### 3.1 是否可以使用 HTTPClient 的实现就可以解决问题？
也不行，如果不做上面的实现，直接改用HTTPClient实现的话，也只是在发送的过程中起到作用，还是需要在前进行处理。

## 4. 参考文献
https://spring.hhui.top/spring-blog/2019/03/27/190327-Spring-RestTemplate之urlencode参数解析异常全程分析/

https://www.w3school.com.cn/tags/html_ref_urlencode.html

