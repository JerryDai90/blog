# 使用 feign 调用服务时，Post 变 Get 请求

我现在使用的是 2.1.1 版本的 feign，进过大量的测试，无论是标准是 `@PostMapping` 还是 `@GetMapping`，只要参数标注 `@RequestParam`，一律都用 `Get` 请求，也就是说把参数拼接到 `URL` 上。经过debug和实验，为了不修改spring boot 自身的代码，寻找到了扩展点。上代码


**增加feign的拦截器 `FeignRequestInterceptor` 即可。**

```java
import feign.Request;
import feign.RequestInterceptor;
import feign.RequestTemplate;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.collections.MapUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.net.URLEncoder;
import java.nio.charset.Charset;
import java.util.*;

import static org.springframework.http.MediaType.APPLICATION_FORM_URLENCODED_VALUE;

@Slf4j
@Component
public class FeignRequestInterceptor implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate template) {

        //目前只支持POST ，表单form提交就会标记（Content-Type: application/x-www-form-urlencoded）
        if ("POST".equals(template.method()) && template.requestBody().length() == 0 && !template.queries().isEmpty()) {
            Object object = MapUtils.getObject(template.headers(), "Content-Type");
            if (null != object) {

                if (((Collection) object).contains(APPLICATION_FORM_URLENCODED_VALUE)) {


                    //TODO 这里参数评价未完善，
                    StringBuilder builder = new StringBuilder();
                    Iterator<String> iterator = template.queries().keySet().iterator();

                    while (iterator.hasNext()) {
                        String next = iterator.next();
                        Collection<String> strings = template.queries().get(next);
                        builder.append(next + "=" + URLEncoder.encode(strings.toString()));
                    }

                    template.body(Request.Body.encoded(builder.toString().getBytes(), Charset.forName("UTF-8")));
                    template.queries(null);
                }
            }
        }

        log.debug("FeignRequestInterceptor:{}", template.toString());
    }
}
```



