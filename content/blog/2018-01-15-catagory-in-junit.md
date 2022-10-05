---
layout: post
title:  "Junit 多策略测试切换"
date:   2018-01-15 20:44:18
categories: java
---

## 介绍

最近在弄单元测试，之前基础版本的测试基本都是写好的，但是缺少高级版本的测试，又不想把代码copy一份，要就地实现基础版和高级版的测试复用，并且需要从配置读取测试策略来指定当前测试策略。因为基础版和高级版只是大部分功能差不多，因此还要做到通过注解的方式来限定当前测试方法或测试类是否运行于当前测试策略。下面废话少说，Let's Go!

## 实践

之前也没有好好深入junit，仔细想了一下，我们要做到以下两点：

1.需要做到通过注解限定当前测试类或测试方法是否执行。这个使用Category注解可以实现，但是Category的作用太过于简单，仅仅只能做到限定测试是否执行。

2.需要做到针对同一测试方法在不同测试策略下可以有不相同的处理，其中可能请求地址不一样，返回结果以及断言处理不一样等。这个之前最初考虑的时候太过于复杂，当时想实现自定义注解，然后通过aop拦截去改写请求参数和处理流程。这样做太过于复杂，而且不同测试策略不会同时运行。最后的做法就是将测试流程提取到接口，各自的测试策略自己去实现，然后在初始化时，根据当前测试策略去绑定对应的实例，测试使用时直接使用接口就可以实现多策略测试的切换，再结合第一点基本上就可以满足需求了。


主要代码如下：

```java
package com.lux.junit;

import com.lux.junit.category.BasicCategory;
import com.lux.junit.category.SeniorCategory;
import com.lux.junit.helper.BaseUserHelper;
import org.junit.Test;
import org.junit.experimental.categories.Category;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class JunitApplicationTests {

    @Autowired
    private BaseUserHelper userHelper;

    @Test
    public void contextLoads() {
    }

    @Test
    @Category({BasicCategory.class, SeniorCategory.class})
    public void getUserTest() {
        System.out.println(userHelper.getUser());
    }

    @Test
    @Category(SeniorCategory.class)
    public void getSeniorUserTest() {
        System.out.println(userHelper.getUser());
    }

}

```



```Java
package com.lux.junit.configure;

import com.lux.junit.category.BasicCategory;
import com.lux.junit.helper.BaseUserHelper;
import com.lux.junit.helper.BasicUserHelper;
import com.lux.junit.helper.SeniorUserHelper;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Created with IntelliJ IDEA.
 * User: chenfeilong
 * Date: 2018/1/28
 * Time: 23:30
 * Description:
 */
@Configuration
public class CategoryConfigure {
    @Value("${test.category}")
    String currentCategory;

    @Bean("userHelper")
    public BaseUserHelper userHelper() {
        if (BasicCategory.class.getName().equals(currentCategory)) {
            return new BasicUserHelper();
        } else {
            return new SeniorUserHelper();
        }
    }
}

```



## 最后

Category需要传递includeCategories参数，其实它可以是多个策略。通常使用命令行传入或使用gradle传入。但是我们需要从配置文件读入，需要像下面这样做。

```groovy
test {
    useJUnit {
        includeCategories getCategory()
    }
}

String getCategory() {
    def category = doGetCategory()
    println("current catagory: " + category)
    return category
}

String doGetCategory() {
    def propertiesFilePath = "src/main/resources/application.properties"
    def defaultCategory = "com.lux.junit.category.BasicCategory"
    File propFile = new File(propertiesFilePath)
    if (propFile.canRead()) {
        Properties props = new Properties()
        props.load(new FileInputStream(propFile))
        return props.getProperty("test.category")
    }
    println("can't find catagory config,use default catagory " + defaultCategory)
    return defaultCategory
}
```

测试结果

BasicCategory:





## 参考目录

1.[junit wiki](https://github.com/junit-team/junit4/wiki/Categories)

2.[完整代码](https://git.coding.net/uleaf/junit.git)