　　整体的请求过程

　　![w400](http://img.lsof.fun/2020-03-10-15838506471882.jpg)

　　调试的时候，发现拿到的 `Scheme` 总是不对

```java
request.getScheme() //总是 http，而不是实际的http或https
request.isSecure() //总是false(因为总是http)

//总是 nginx 请求的 IP，而不是用户的IP request.getRequestURL() 
//总是 nginx 请求的URL 而不是用户实际请求的 URL response.sendRedirect( 相对url ) //总是重定向到 http 上 (因为认为当前是 http 请求)
request.getRemoteAddr() 
```

　　**问题处理办法**
1、 nginx 增加配置:

```nginx
proxy_redirect http:// $scheme://;
proxy_set_header X‐Forwarded‐Proto $scheme;
proxy_set_header Host $host;
proxy_set_header X‐Real‐IP $remote_addr;
proxy_set_header X‐Forwarded‐For $proxy_add_x_forwarded_for;
```

　　2、 Tomcat server.xml 的 Engine 模块下配置一个 Valve(注意是 Valve，不是 Value， 是 V 不是 U):

```xml
<Engine name="Catalina" defaultHost="localhost">
  <Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
    <Valve className="org.apache.catalina.valves.RemoteIpValve"
         remoteIpHeader="X‐Forwarded‐For"
         protocolHeader="X‐Forwarded‐Proto"
         protocolHeaderHttpsValue="https" />
  </Host>
</Engine>
```

　　附 RemoteIpValve 官方文档:[http://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/valves/RemoteIpValve.html](http://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/valves/RemoteIpValve.html)

　　
