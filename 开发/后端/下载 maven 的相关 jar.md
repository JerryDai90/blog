　　转自：https://www.ktanx.com/blog/p/4033

　　经常碰到这种事情:

　　在一些非 maven 工程中(由于某种原因这种工程还是手工添加依赖的),需要用到某个新的类库(假设这个类库发布在 maven 库中),而这个类库又间接依赖很多其他类库,如果依赖路径非常复杂的话,一个个检查手动下载是很麻烦的事.

　　下面给出一个便捷的办法:

　　创建一个新目录里面建一个 maven pom 文件, 添加需要依赖的类库:

```
<?xml version="1.0"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.dep.download</groupId>
    <artifactId>dep-download</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>com.xx.xxx</groupId>
            <artifactId>yy-yyy</artifactId>
            <version>x.y.z</version>
            <scope/>
        </dependency>
    </dependencies>
</project>
```

　　在这个目录下运行命令:

```shell
mvn -f download-dep-pom.xml dependency:copy-dependencies
```

　　所有跟这个类库相关的直接和间接依赖的 jar 包都会下载到 ./target/dependency/下

　　
