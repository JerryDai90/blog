# Spring IOC 简单实现

## 1、实现说明
本次实现的是一个简单版的spring IOC，仅仅对构造函数和成员变量进行自动注入实现。我重新画了实现图（基本原理和Spring的一致的）。

![w400](http://img.lsof.fun/2020-02-02-IMG_FB497C45B8AC-1.jpeg)


## 2、代码
下面实现的代码有好多地方不严谨，只是实现了功能而已。

源码地址：https://github.com/JerryDai90/java-case/tree/master/spring/ioc/src/main/java/fun/lsof/spring/ioc/simulation

大概说明：

* /support/AnnotationApplicationContext.java ： 解析对象并且进行注入类
* /annotation/Autowired.java ：注解
* /test/MainTest.java ： 测试类


