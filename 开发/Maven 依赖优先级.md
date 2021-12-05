由于对maven依赖配置的优先级不确定，所以通过下面的实验来证明相关的配置

## 1. 准备工作

### 1.1 **配置setting.xml**

　　配置 profile

```xml
<!-- 配置profile 2个资源地址 -->
</profiles>
    <profile>
        <id>pf1</id>
        <repositories>
            <repository>
                <id>profile1</id>
                <name>test name</name>
                <url>http://profile1/</url>
                <!-- <layout>legacy</layout> -->
            </repository>
            <repository>
                <id>profile2</id>
                <name>test name</name>
                <url>http://profile2/</url>
            </repository>
        </repositories>
    </profile>
</profiles>

<activeProfiles>
    <!--激活 -->
    <activeProfile>pf1</activeProfile>
</activeProfiles>
```

　　配置 mirror

```
<mirror>
    <id>mirror1</id>
    <name>aliyun maven</name>
    <url>http://mirror1/</url>
    <mirrorOf>central</mirrorOf>
</mirror> 
```

### 1.2 配置项目的pom.xml

```xml
<repositories>
    <repository>
        <id>pom1</id>
        <url>https://pom1/</url>
        <layout>default</layout>
    </repository>
    <repository>
        <id>pom2</id>
        <url>https://pom2/</url>
        <layout>default</layout>
    </repository>
</repositories>
```

## 2. 试验

　　运行`mvn clean install` 后输出

```
$ mvn clean install                                                                                                                                                                                                                                                1 ↵
[INFO] Scanning for projects...
Downloading: http://profile1/org/springframework/boot/spring-boot-starter-parent/2.5.5/spring-boot-starter-parent-2.5.5.pom
Downloading: http://profile2/org/springframework/boot/spring-boot-starter-parent/2.5.5/spring-boot-starter-parent-2.5.5.pom
Downloading: https://pom1/org/springframework/boot/spring-boot-starter-parent/2.5.5/spring-boot-starter-parent-2.5.5.pom
Downloading: https://pom2/org/springframework/boot/spring-boot-starter-parent/2.5.5/spring-boot-starter-parent-2.5.5.pom
Downloading: http://mirror1/org/springframework/boot/spring-boot-starter-parent/2.5.5/spring-boot-starter-parent-2.5.5.pom
[ERROR] [ERROR] Some problems were encountered while processing the POMs:
```

　　看到输出得 setting.xml profile1/profile1 > 项目pom.xml pom1/pom2 > setting.xml mirror1，由此可得出以下结论

* setting.xml profile设置后，即为优先级最高。
* 前面找不到才会向下寻找。

　　部分实验结果过程不再赘述，直接说结论

* pom.xml repositories中定义的远程仓库ID可以被setting.xml profile 覆盖。其实pom.xml repositories 和 profile/repositories 作用一致，只是setting优先级高
* 如果setting.xml mirror 如果设置mirrorOf为*，则代理所有（忽略profile、pom中的设置）
* pom.xml repositories中定义的私服，只能是当前文件定义的依赖下载使用。层级不传递。

### 2.1 业务需求实践

* 如果需要编写给到第三方使用的工具类，可以不将jar上传到中央仓库。可以在自己的pom文件中定义repositories，默认会先从你指定的仓库下载。如果找不到的资源才到从其他指定的仓库下载。（注意：一定是setting.xml 中 mirrorOf不能设置为\*，如果设置为\*则直接代理了全部。）
