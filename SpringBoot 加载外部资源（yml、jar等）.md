# SpringBoot 加载外部资源（yml、jar等）

## 1. 需求

由于 SpringBoot 打包后，默认是不能加载外部的jar文件，只能默认加载 yml 文件。

> 想通过外部的 jar 来扩展此微服务的能力，而且主 jar 升级更新不受影响。这种方式适用于已经有了底座服务。但是底座服务不满足现有需求，可以通过外部的 jar 来扩展相关业务。

## 2. 实现

由于SpringBoot 默认启动类是 `org.springframework.boot.loader.JarLauncher`, 具体看查看打包后的jar 中的 META-INF/xxx/MANIFEST.MF

```
Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Built-By: jerry
Start-Class: lsof.fun.test.MainApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Version: 2.2.8.RELEASE
Created-By: Apache Maven 3.3.9
Build-Jdk: 1.8.0_60
Main-Class: org.springframework.boot.loader.JarLauncher
```

而 `JarLauncher` 是无法配置相关外部依赖环境，需要更换为 `PropertiesLauncher`，因此需要修改打包配置，增加 layout 与finalName 相关配置，如下：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
            <configuration>
                <finalName>auth-test</finalName>
                <layout>ZIP</layout>
            </configuration>
        </execution>
    </executions>
</plugin>
```

启动脚本：

```
java -Dloader.path=$PATH -jar auth-test.jar
```

* $PATH：可以是相对路径可以是绝对路径，可以是具体的jar或者文件夹

执行后即可加载相关文件到 classpath了，如果jar中存在自动装配类，也会自动加载。

## 3. 相关资料
https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-executable-jar-format.html#executable-jar-property-launcher-features



