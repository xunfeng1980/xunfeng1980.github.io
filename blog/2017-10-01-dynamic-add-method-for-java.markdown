---
layout: post
title:  "Java 动态增加方法"
date:   2017-10-01 23:44:18
categories: jdk
---

```java
package com.lux.study.assist;

import javassist.*;

/**
 * @author: lux
 * @date: 2017/10/25 14:54
 */
public class App {
    public static void main(String[] args) {
        try {
            String className = "com.lux.study.assist.UserInfo";
            UserInfo userInfo = new UserInfo();
            userInfo.setName("test");
            userInfo.setId(1);
            System.out.println("before:" + userInfo);
            ClassPool pool = ClassPool.getDefault();
            CtClass cc = pool.get(className);
            CtMethod mthd = CtNewMethod.make("public String test() { return \"test() is called \"+ toString();  }", cc);
            cc.addMethod(mthd);

            AppClassLoader appClassLoader = AppClassLoader.getInstance();
            Class<?> clazz = appClassLoader.findClassByBytes(className, cc.toBytecode());
//            clazz.getDeclaredConstructor().newInstance();
            Object obj = appClassLoader.getObj(clazz,userInfo);
            System.out.println("after:" + obj);
            //测试反射调用添加的方法
            System.out.println(obj.getClass().getDeclaredMethod("test").invoke(obj));

        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}

```



```java
package com.lux.study.assist;


import java.lang.reflect.Field;

/**
 * @author: lux
 * @date: 2017/10/24 13:48
 */
public class AppClassLoader extends ClassLoader {

    private static class SingletonHolder {
        public final static AppClassLoader instance = new AppClassLoader();
    }

    public static AppClassLoader getInstance() {
        return SingletonHolder.instance;
    }


    private AppClassLoader() {

    }

    /**
     * 通过classBytes加载类
     *
     * @param className
     * @param classBytes
     * @return
     */
    public Class<?> findClassByBytes(String className, byte[] classBytes) {
        return defineClass(className, classBytes, 0, classBytes.length);
    }

    /**
     * 复制对象所有属性值,并返回一个新对象
     *
     * @param srcObj
     * @return
     */
    public Object getObj(Class<?> clazz, Object srcObj) {
        try {
            Object newInstance = clazz.getDeclaredConstructor().newInstance();
            Field[] fields = srcObj.getClass().getDeclaredFields();
            for (Field oldInstanceField : fields) {
                String fieldName = oldInstanceField.getName();
                oldInstanceField.setAccessible(true);
                Field newInstanceField = newInstance.getClass().getDeclaredField(fieldName);
                newInstanceField.setAccessible(true);
                newInstanceField.set(newInstance, oldInstanceField.get(srcObj));
            }
            return newInstance;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}

```



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

```shell
before:UserInfo{id=1, name='test'}
after:UserInfo{id=1, name='test'}
test() is called UserInfo{id=1, name='test'}
```

借鉴了spring devtool的热部署实现，通过使用javassist或者cglib来实现字节码的生成，然后通过自定义的类加载器加载修改之后的类，最后使用反射将属性值拷贝过来，就可以得到一个和之前看起来差不多的类，但是却有我们自定义方法的对象。