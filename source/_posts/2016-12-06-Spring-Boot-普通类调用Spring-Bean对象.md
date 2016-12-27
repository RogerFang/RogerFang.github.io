---
title: 'Spring Boot:普通类调用Spring Bean对象'
date: 2016-12-06 14:19:06
categories:
- Spring Boot
---

编写的SpringUtil在Spring Boot可以扫描的包下或者使用@ComponentScan引入自定义的包了，只需要使得SpringUtil实现接口：ApplicationContextAware，然后加上@Component 注解即可。

```java
package com.roger.util;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * 普通类使用Spring Bean对象
 */
@Component
public class SpringUtil implements ApplicationContextAware {
    private static ApplicationContext ctx = null;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if (ctx == null){
            ctx = applicationContext;
        }
    }

    /**
     * 获取ApplicationContext
     * @return
     */
    public static ApplicationContext getApplicationContext(){
        return ctx;
    }

    /**
     * 通过Bean name获取Bean
     * @param name
     * @return
     */
    public static Object getBean(String name){
        return ctx.getBean(name);
    }

    /**
     * 通过Bean class获取Bean
     * @param clazz
     * @param <T>
     * @return
     */
    public static <T> T getBean(Class<T> clazz){
        return ctx.getBean(clazz);
    }

    /**
     * 通过Bean name以及Class返回指定的Bean
     * @param name
     * @param clazz
     * @param <T>
     * @return
     */
    public static <T> T getBean(String name, Class<T> clazz){
        return ctx.getBean(name, clazz);
    }
}
```

