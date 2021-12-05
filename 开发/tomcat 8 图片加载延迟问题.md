　　此问题在 tomcat7 没问题，经测试验证后 tomcat8 由此问题。

## 问题描述

　　动态上传、生成图片的时候，立即访问此图片的时候会出现访问不到，过几秒中之后才可以访问到图片。

## 解决

　　在此文件增加 $tomcat_home/conf/context.xml里面的 Context 中增加代码

```
<Resources cachingAllowed="false" /> 
```

　　context.xml 示例如下：

```xml
<Context>
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>
    <!-- 增加此行代码 -->
    <Resources cachingAllowed="false" />
</Context>
```

## 备注

　　参数说明
![](http://img.lsof.fun/2018-05-25-15248197821035.jpg)

　　附上地址 [https://tomcat.apache.org/tomcat-8.0-doc/config/resources.html](https://tomcat.apache.org/tomcat-8.0-doc/config/resources.html)
