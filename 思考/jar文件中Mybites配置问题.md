　　Mybites 配置在classpath中是可以加载的，本文Q&A主要解jar包中加载Mybites问题

#### 1. *Mapper.xml 读取不到

　　需要配置 `yml` 中 `mybatis-plus.mapper-locations`，改成 `classpath*`，样例如下

```
mybatis-plus:
  mapper-locations: classpath*:mapper/*.xml
```

#### 2. 接口与实现类重复注册到Bean容器中

　　主要是由于 `@MapperScan` 问题，将所有的接口都标记为Mapper 接口，所以导致有同一个代理类

　　**解决办法**
@MapperScan 增加 markerInterface接口，Mapper 的父类即可。

```
@MapperScan(basePackages = "lsof.fun", markerInterface = BaseMapper.class)
```

### 3. 参考

　　https://www.javatt.com/p/80860
https://segmentfault.com/a/1190000012470056

　　
