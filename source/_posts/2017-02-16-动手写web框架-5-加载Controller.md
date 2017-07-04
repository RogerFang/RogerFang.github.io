---
title: '动手写web框架(5): 加载Controller'
date: 2017-02-16 15:33:12
categories:
- 动手写web框架
---

通过**ClassHelper**可以获取所有定义了`@Controller`注解的类，通过**反射**获取该类中所有带有`@Action`注解的方法，获取Action注解中的请求表达式，进而得到请求方法和请求路径，将其封装成一个请求对象`Request`和处理对象`Handler`，最后把`Request`和`Handler`建立映射关系，并提供一个可根据请求方法与请求路径获取处理对象的方法。

```java Request
package org.smart4j.framework.bean;

import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;

/**
 * 封装请求信息
 * Created by Roger on 2016/11/23.
 */
public class Request {

    /**
     * 请求信息
     */
    private String requestMethod;

    /**
     * 请求方法
     */
    private String requestPath;

    public Request(String requestMethod, String requestPath) {
        this.requestMethod = requestMethod;
        this.requestPath = requestPath;
    }

    public String getRequestMethod() {
        return requestMethod;
    }

    public String getRequestPath() {
        return requestPath;
    }

    @Override
    public int hashCode() {
        return HashCodeBuilder.reflectionHashCode(this);
    }

    /**
     * 两个对象相等当且仅当每个属性值都相等
     * @param obj
     * @return
     */
    @Override
    public boolean equals(Object obj) {
        return EqualsBuilder.reflectionEquals(this, obj);
    }
}
```

```java Handler
package org.smart4j.framework.bean;

import java.lang.reflect.Method;

/**
 * 封装Action信息
 * Created by Roger on 2016/11/23.
 */
public class Handler {

    /**
     * Controller类
     */
    private Class<?> controllerClass;

    /**
     * Action方法
     */
    private Method actionMethod;

    public Handler(Class<?> controllerClass, Method actionMethod) {
        this.controllerClass = controllerClass;
        this.actionMethod = actionMethod;
    }

    public Class<?> getControllerClass() {
        return controllerClass;
    }

    public Method getActionMethod() {
        return actionMethod;
    }
}
```

```java ControllerHelper
package org.smart4j.framework.helper;

import org.apache.commons.collections4.CollectionUtils;
import org.apache.commons.lang3.ArrayUtils;
import org.smart4j.framework.annotation.Action;
import org.smart4j.framework.bean.Handler;
import org.smart4j.framework.bean.Request;

import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

/**
 * 封装请求对象(Request)与处理对象(Handler)的映射关系
 * Created by Roger on 2016/11/23.
 */
public final class ControllerHelper {

    private static final Map<Request, Handler> ACTION_MAP = new HashMap<>();

    static {
        Set<Class<?>> controllerClassSet = ClassHelper.getControllerClassSet();
        if (CollectionUtils.isNotEmpty(controllerClassSet)){
            for (Class<?> controllerCls: controllerClassSet){
                // 获取Controller类中定义的方法
                Method[] methods = controllerCls.getDeclaredMethods();

                for (Method method: methods){
                    if (method.isAnnotationPresent(Action.class)){
                        // 从Action中获取URL映射规则
                        Action action = method.getAnnotation(Action.class);
                        String mapping = action.value();
                        if (mapping.matches("\\w+:/\\w*")){
                            String[] array = mapping.split(":");
                            if (ArrayUtils.isNotEmpty(array) && array.length == 2){
                                String requestMethod = array[0];
                                String requestPath = array[1];
                                Request request = new Request(requestMethod.toLowerCase(), requestPath);
                                Handler handler = new Handler(controllerCls, method);
                                ACTION_MAP.put(request, handler);
                            }
                        }
                    }
                }
            }
        }
    }

    public static Handler getHandler(String requestMethod, String requestPath){
        Request request = new Request(requestMethod.toLowerCase(), requestPath);
        return ACTION_MAP.get(request);
    }
}

```