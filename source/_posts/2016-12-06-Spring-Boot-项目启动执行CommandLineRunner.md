---
title: 'Spring Boot:项目启动执行CommandLineRunner'
date: 2016-12-06 15:30:23
categories:
- Spring Boot
---

Spring Boot提供了一个借口CommandLineRunner，实现这个接口可以使得在项目启动时去执行一些操作。@Order注解来规定调用所有CommandLineRunner实例的顺序。

```java
package com.roger.commandline;

import org.springframework.boot.CommandLineRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Order(value = 1)
public class FirstCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... strings) throws Exception {
        System.out.println(">>>>>>>>first command line runner: " + strings);
    }
}
```

