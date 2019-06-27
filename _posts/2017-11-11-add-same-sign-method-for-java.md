---
layout: post
title:  "添加签名相同但返回值不同的方法"
date:   2017-11-01 23:44:18 +0800
categories: java
---


在面试时，被问到，为什么重载是参数不相同，而不是返回值不相同或者同时不相同？仔细一想，这个问题意义并不大，我们来做一个实验。

```java
package com.lux.study.assist;

/**
 * @author: lux
 * @date: 2017/10/24 13:51
 */
public class UserInfo {
    private Integer id;
    private String name;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "UserInfo{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}

```

这是上次的主角UserInfo类，如果我们尝试添加一个返回值为Integer的getName方法，将无法通过编译。

```shell
Error:(22, 20) java: 已在类 com.lux.study.assist.UserInfo中定义了方法 getName()
```

下面我们通过javassist来生成这样一个签名相同返回值不同的函数，并反射调用。

```java
package com.lux.study.assist;

import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;
import javassist.CtNewMethod;

import java.lang.reflect.Method;

/**
 * @author: lux
 * @date: 2017/11/1 23:25
 */
public class MethodTest {
    public static void main(String[] args) {
        String className = "com.lux.study.assist.UserInfo";
        UserInfo userInfo = new UserInfo();
        userInfo.setName("test");
        userInfo.setId(1);
        try {
            ClassPool pool = ClassPool.getDefault();
            CtClass cc = pool.get(className);
            CtMethod mthd = CtNewMethod.make("public Integer getName() { return  Integer.valueOf(2); }", cc);
            cc.addMethod(mthd);
            AppClassLoader appClassLoader = AppClassLoader.getInstance();
            Class<?> clazz = appClassLoader.findClassByBytes(className, cc.toBytecode());
            Object obj = appClassLoader.getObj(clazz, userInfo);
            Method[] methods = obj.getClass().getDeclaredMethods();
            for (Method method : methods) {
                if (method.getName().contains("getName")) {
                    System.out.println(method.toString() + " -> " + method.invoke(obj));
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```



这是最后的结果，证明方法返回值还是有用的，至于为什么重载的时候，必须是参数不相同，而不是返回值不相同，可能就是别人说的，大多时候，我们可能并不关心返回结果。

```shell
public java.lang.Integer com.lux.study.assist.UserInfo.getName() -> 2
public java.lang.String com.lux.study.assist.UserInfo.getName() -> test
```

