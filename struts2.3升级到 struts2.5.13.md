# struts2.3升级到 struts2.5.13
# 1. log4j 1 升级到 log4j 2

1、删除掉 log4j 1的配置文件。如 classpath 下面的 log4j.xml 或者 log4j.properties。
2、新增 log4j2.xml 文件在 classpath 下。以下是最基本的配置。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="STDOUT" target="SYSTEM_OUT">
            <PatternLayout pattern="%d %-5p [%t] %C{2} (%F:%L) - %m%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <!-- all 为所有的都输出 -->
        <Root level="all">
            <AppenderRef ref="STDOUT"/>
        </Root>
    </Loggers>
</Configuration>
```

我的生产配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="STDOUT" target="SYSTEM_OUT">
            <PatternLayout pattern="%d %-5p [%t] %C{2} (%F:%L) - %m%n"/>
        </Console>

        <RollingFile name="RollingFileInfo" fileName="${sys:catalina.home}/logs/web/info.log"
                     filePattern="${sys:catalina.home}/logs/web/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
            <Filters>
                <!--设置只输出级别为INFO的日志-->
                <ThresholdFilter level="INFO"/>
                <ThresholdFilter level="WARN" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="ERROR" onMatch="DENY" onMismatch="NEUTRAL"/>
            </Filters>
            <PatternLayout pattern="%d %-5p [%t] %C{2} (%F:%L) - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="20MB"/>
            </Policies>
        </RollingFile>

        <RollingFile name="RollingFileWarn" fileName="${sys:catalina.home}/logs/web/warn.log"
                     filePattern="${sys:catalina.home}/logs/web/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log">
            <Filters>
                <!--设置只输出级别为WARN的日志-->
                <ThresholdFilter level="WARN"/>
                <ThresholdFilter level="ERROR" onMatch="DENY" onMismatch="NEUTRAL"/>
            </Filters>
            <PatternLayout pattern="%d %-5p [%t] %C{2} (%F:%L) - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="20MB"/>
            </Policies>
        </RollingFile>

        <RollingFile name="RollingFileError" fileName="${sys:catalina.home}/logs/web/error.log"
                     filePattern="${sys:catalina.home}/logs/web/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log">
            <Filters>
                <!--设置只输出级别为ERROR的日志-->
                <ThresholdFilter level="ERROR"/>
            </Filters>
            <PatternLayout pattern="%d %-5p [%t] %C{2} (%F:%L) - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="20MB"/>
            </Policies>
        </RollingFile>
    </Appenders>
    <Loggers>
        <!--<logger name="org.springframework" level="INFO"/>-->
        <!--<logger name="org.hibernate" level="INFO"/>-->
        <Root level="info">

            <AppenderRef ref="STDOUT"/>
            <AppenderRef ref="RollingFileInfo"/>
            <AppenderRef ref="RollingFileWarn"/>
            <AppenderRef ref="RollingFileError"/>
        </Root>
    </Loggers>
</Configuration>
```

3、maven 配置

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.8.2</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.8.2</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-web</artifactId>
    <version>2.8.2</version>
</dependency>
```

# 2. 去除xerces依赖
1、 移除对 xerces 的依赖包括 xerces、xercesImpl、mlParserAPIs（重点）
2、 maven 移除以下配置（检查 lib 中是否已经移除掉了）

```xml
<dependency>
    <groupId>org.apache</groupId>
    <artifactId>xerces</artifactId>
    <version>2.5.0</version>
</dependency>
<dependency>
    <groupId>xerces</groupId>
    <artifactId>xercesImpl</artifactId>
    <version>2.10.0</version>
</dependency>
```
maven 工程，需要分析依赖（使用命令 `mvn dependency:tree` ）是否存在传递的依赖的xerces
[阻止依赖传送门](http://chwshuang.iteye.com/blog/2069937) 
注意：如果依赖的 jar 文件中还存在以上的 jar 会出现一个错误是说注解无法注入。据说是 JDK 默认携带了此类的实现。

>小插曲：出现下面的问题的时候，真心奔溃。这个提示明显是说annotation转换出现异常。看了2.5.13的源码。`org.apache.struts2.convention.annotation.Result.name()`这个是一个数组，而在2.3里面确实一个`string`，各种排查。无果。重新`build`了工程，也就是去掉了`xerces`这些依赖
>
```
java.lang.annotation.AnnotationTypeMismatchException: Incorrectly typed data found for annotation element public abstract java.lang.String[] org.apache.struts2.convention.annotation.Result.name() (Found data of type class java.lang.String[view])
```

# 3. Struts2 配置文件修改
struts2 的配置文件修改 dtd 头部引用

```xml
<!DOCTYPE struts PUBLIC
	"-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
	"http://struts.apache.org/dtds/struts-2.5.dtd">
```
web.xml修改

```xml
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>
        org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter
    </filter-class>
</filter>
```

# 4. 遗留问题
如果系统中使用 struts `struts.convention.action.includeJars`来导入包的话（配置如下）。会在tomcat6和7偶然出现在 classpath 找不到指定的 jar 文件。但是在最新版的 tomcat 8.5.23 是可以的。但是最奇怪的是 `build` 多几次就可以了 ~_~

```xml
<constant name="struts.convention.action.includeJars" value=".*?/includeFile.*?jar(!/)?" />
```
**后面要研究一下这个扫包的逻辑**

# 4.1 更新20171126
经过发现 `struts.convention.action.includeJars` 的 `jar` 文件没有`META-INF`文件夹。正确的jar 文件目录是

```
*.jar
|--META-INF
|--com
    |--youpackage
```
而在 lib 下面的却没有 `META-INF`。
使用 命令`mvn clean install` 打包的时候看了输出的目录的 `jar` 所有东西都有。然后在本地的 `repositories` 里面的 jar 也没有问题。

![-c](http://img.lsof.fun/2019-01-28-15486498815104.jpg)
后来发现原来是这个锅是 `IDEA` 的。系统分为4个工程。依赖如下

```
d工程（web）
|--src
|--WebRoot
    |--WEB-INF
        |--lib
            |--a工程.jar
            |--b工程.jar
            |--c工程.jar
        |--classes 
```
如果在 `IDEA` 中导入 d工程。然后再导入 其他三个工程。启动 tomcat 的时候 IDEA 会自动编译一下其他的三个工程。把其他三个工程的 `jar` 文件打包到 `lib` 下面。打包的时候就出现没有 `jar` 文件 `META-INF`这个目录。最奇怪的第二次启动就可以了~_~。

# 5. 备注
1. 实验过 `jar` 文件中必须有`META-INF`这层目录（里面可以没有描述文件）。没有的话跳过此包不扫描。*（猜想：估计在 tomcat6、7 扫描的时候没有此目录的话可能会被认为是普通的压缩文件。不需要装载到系统中去。tomcat8是支持没有`META-INF`的）* 。有兴趣的可以去官网看卡 [传送门](https://docs.oracle.com/javase/7/docs/technotes/guides/jar/jar.html)。另外发现IBM 一篇干货 [link](https://www.ibm.com/developerworks/cn/java/j-jar/index.html)



