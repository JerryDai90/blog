# Spring IOC 简单实现

## 1、实现说明
本次实现的是一个简单版的spring IOC，仅仅对构造函数和成员变量进行自动注入实现。我重新画了实现图（基本原理和Spring的一致的）。

![IMG_FB497C45B8AC-1](http://img.lsof.fun/2020-02-02-IMG_FB497C45B8AC-1.jpeg)


## 2、代码
下面实现的代码有好多地方不严谨，只是实现了功能而已。

### 2.1、Autowired4IOC 注解类
```java
package com.ioc4dyc.annotation;
import java.lang.annotation.*;

@Target({ElementType.FIELD, ElementType.CONSTRUCTOR, ElementType.METHOD})
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Autowired4IOC {
}
```

### 2.2、ApplicationContext4IOC 具体获取对象类
```java
import com.ioc4dyc.annotation.Autowired4IOC;

import java.lang.annotation.Annotation;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class ApplicationContext4IOC {

    /**
     * 初始化成功后暴露到这里（最小初始化，和spring IOC 实现的原理一致）
     */
    static ThreadLocal<Map<String, Object>> discover = new ThreadLocal<Map<String, Object>>();

    /**
     * 判断构造函数是否有自动注入注解
     */
    private static boolean hasConstructorAutowired(Class<?> aClass) {
        Constructor<?>[] constructors = aClass.getConstructors();
        for (Constructor _c : constructors) {
            Annotation annotation = _c.getAnnotation(Autowired4IOC.class);
            if (null != annotation) {
                return true;
            }
        }
        return false;
    }

    /**
     * 解析构造函数.
     */
    private static Object parseConstructor(Class<?> aClass) throws Exception {
        Object obj = null;

        Constructor<?>[] constructors = aClass.getConstructors();

        for (Constructor _c : constructors) {

            List<Object> params = new ArrayList<Object>();

            Annotation annotation = _c.getAnnotation(Autowired4IOC.class);

            if (null != annotation) {
                Class[] parameterTypes = _c.getParameterTypes();

                for (Class c : parameterTypes) {
                    params.add(getBean(c));
                }

                obj = _c.newInstance(params.toArray());
                break;
            }
        }
        return obj;
    }

    /**
     * 解析成员变量.
     */
    private static void parseFields(Object obj, Field[] declaredFields) throws IllegalAccessException {
        for (Field f : declaredFields) {
            Autowired4IOC annotation = f.getAnnotation(Autowired4IOC.class);
            if (null != annotation) {
                Object fObj = getBean(f.getType());
                f.setAccessible(true);
                f.set(obj, fObj);
            }
        }
    }


    /**
     * 获取对象实例.
     */
    public static <T> T getBean(Class<T> t) {
        Map<String, Object> threadData = discover.get();
        if (null == threadData) {
            discover.set(new HashMap<String, Object>());
            threadData = discover.get();
        }

        try {
            String name = t.getName();

            if (threadData.containsKey(name)) {
                return (T) threadData.get(name);
            }

            Class<?> aClass = Class.forName(name);
            Object obj = null;

            //优先判断构造函数
            if (hasConstructorAutowired(aClass)) {
                //构造函数注入
                obj = parseConstructor(aClass);
            } else {
                //先暴露出去
                obj = aClass.newInstance();
            }
            threadData.put(name, obj);

            //成员变量
            Field[] declaredFields = aClass.getDeclaredFields();
            if (declaredFields.length != 0) {
                parseFields(obj, declaredFields);
            }

            return (T) obj;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

### 2.2、测试类

#### 2.2.1 测试A

```java
package com.ioc4dyc.testioc;

import com.ioc4dyc.annotation.Autowired4IOC;

public class A {

    @Autowired4IOC
    private B b;

    @Autowired4IOC
    private C c;

    public void say(){
        System.out.println(b.toString());
        System.out.println(c.toString());

        b.say();
        c.say();
    }
}

```
#### 2.2.2 测试B

```java
package com.ioc4dyc.testioc;

import com.ioc4dyc.annotation.Autowired4IOC;

public class B {

    @Autowired4IOC
    C c;

    public void say() {
        System.out.println("class: " + this.toString());
    }

}

```
#### 2.2.3 测试C

```java
package com.ioc4dyc.testioc;

import com.ioc4dyc.annotation.Autowired4IOC;

public class C {

    @Autowired4IOC
    A a;

    public void say(){
        System.out.println("class: "+this.toString());
    }
}

```

#### 2.2.4 测试D
```java
package com.ioc4dyc.testioc;

import com.ioc4dyc.annotation.Autowired4IOC;

public class D {

    @Autowired4IOC
    C c1;

    @Autowired4IOC
    public D(A a, B b, C c){
        System.out.println(a.toString());
        System.out.println(b.toString());
        System.out.println(c.toString());
    }

    public void say(){
        System.out.println("class: "+this.toString());
        System.out.println("C1："+c1.toString());
    }
}

```

#### 2.2.5 主函数入口

```java
package com.ioc4dyc.testioc;

import com.ioc4dyc.ioc.ApplicationContext4IOC;

public class MainTest {

    public static void main(String[] args) throws InterruptedException {

        A a = ApplicationContext4IOC.getBean(A.class);
        a.say();

        System.out.println("--------------------------------");

        new Thread(new Runnable() {
            @Override
            public void run() {
                D a = ApplicationContext4IOC.getBean(D.class);
                a.say();
            }
        }).start();


        Thread.sleep(100L);
        System.out.println("--------------------------------");

        new Thread(new Runnable() {
            @Override
            public void run() {
                D a = ApplicationContext4IOC.getBean(D.class);
                a.say();
            }
        }).start();

    }


}

```

