---
title: 'Dubbo:dubbo与spring'
date: 2017-12-18 21:08:16
categories:
- dubbo
tags:
- dubbo
---

# spring`<dubbo>`标签扩展
## spring自定义标签
扩展spring自定义标签的步骤如下：
- 定义一个XSD文件描述扩展组件内容。
- 实现`BeanDefinitionParser`接口，用于解析XSD文件中组件的定义，置于类路径下。
- 继承`NamespaceHandlerSupport`类，将组件解析器注册到spring容器中。
- 在类路径下创建 **META-INF/spring.schemas** 文件，spring可扩展的schema机制默认会去加载该文件，作为自定义标签的命名空间的本地映射。
- 在类路径下创建 **META-INF/spring.handlers**文件，spring在`parseCustomElement()`解析自定义标签时会加载对应的`NamespaceHandler`，然后调用其`parse()`对相应的元素节点进行解析。

## dubbo中的实现
`DubboNamespaceHandler`、`DubboBeanDefinitionParser`，根据不同的节点名解析成不同的对象(eg: `ApplicationConfig`、`RegistryConfig`、`ProtocolConfig`、`ProviderConfig`、`ConsumerConfig`、`ReferenceBean`、`ServiceBean`等)。

# 服务暴露触发
![](/images/dubbo/servicebean-class-diagram.png)

`ServiceBean`实现了`InitializingBean`、`ApplicationListener`，对应的会在`afterPropertiesSet()`bean的初始化过程（设置一个非空且不等-1的值，延迟delay毫秒）和`onApplicationEvent`spring容器`refresh()`完成后的事件机制（delay设为 null或-1），判断是否需要export服务。

真正的服务暴露逻辑是在，`ServiceBean`的父类`ServiceConfig.export()`完成服务暴露，暴露之前还需要将bean转换成URL格式（将url作为解耦的通信数据）。

# 服务引用触发
![](/images/dubbo/referencebean-class-diagram.png)

`ReferenceBean`实现了`FactoryBean`、`InitializingBean`，在reference标签中配置init属性就会触发立即初始化（`afterPropertiesSet()`方法中），如果没有配置则会延迟初始化直到通过`getBean()`调用`getObject()`完成。

真正的服务引用逻辑是在，`ReferenceBean`的父类`ReferenceConfig.init()`中完成服务引用。

# SpringContainer
```java SpringContainer
public void start() {
    String configPath = ConfigUtils.getProperty(SPRING_CONFIG);
    if (configPath == null || configPath.length() == 0) {
        configPath = DEFAULT_SPRING_CONFIG;
    }
    context = new ClassPathXmlApplicationContext(configPath.split("[,\\s]+"));
    context.start();
}
```

默认加载`classpath*:META-INF/spring/*.xml`相关的spring配置文件，也可以通过`dubbo.spring.config`参数进行设定。

# Main加载spring容器
dubbo提供了是一个 `SpringContainer` 的容器，以简单的 `com.alibaba.dubbo.container.Main`类来加载 Spring 启动，因为服务通常不需要 Tomcat/JBoss 等 Web 容器的特性，没必要用 Web 容器去加载服务。

通过重入锁和等待条件来保证Spring容器一直运行不退出，通过注册一个shutdown hook来保证JVM关闭时清理现场，释放通知信号。
```java
private static final ReentrantLock LOCK = new ReentrantLock();

private static final Condition STOP = LOCK.newCondition();

public static void main(String[] args) {
    try {
        if (args == null || args.length == 0) {
            String config = ConfigUtils.getProperty(CONTAINER_KEY, loader.getDefaultExtensionName());
            args = Constants.COMMA_SPLIT_PATTERN.split(config);
        }

        final List<Container> containers = new ArrayList<Container>();
        for (int i = 0; i < args.length; i++) {
            containers.add(loader.getExtension(args[i]));
        }
        logger.info("Use container type(" + Arrays.toString(args) + ") to run dubbo serivce.");

        if ("true".equals(System.getProperty(SHUTDOWN_HOOK_KEY))) {
            Runtime.getRuntime().addShutdownHook(new Thread() {
                public void run() {
                    for (Container container : containers) {
                        try {
                            container.stop();
                            logger.info("Dubbo " + container.getClass().getSimpleName() + " stopped!");
                        } catch (Throwable t) {
                            logger.error(t.getMessage(), t);
                        }
                        try {
                            LOCK.lock();
                            STOP.signal();
                        } finally {
                            LOCK.unlock();
                        }
                    }
                }
            });
        }

        for (Container container : containers) {
            container.start();
            logger.info("Dubbo " + container.getClass().getSimpleName() + " started!");
        }
        System.out.println(new SimpleDateFormat("[yyyy-MM-dd HH:mm:ss]").format(new Date()) + " Dubbo service server started!");
    } catch (RuntimeException e) {
        e.printStackTrace();
        logger.error(e.getMessage(), e);
        System.exit(1);
    }
    try {
        LOCK.lock();
        STOP.await();
    } catch (InterruptedException e) {
        logger.warn("Dubbo service server stopped, interrupted by other thread!", e);
    } finally {
        LOCK.unlock();
    }
}
```

shutdown hook 调用场景：
- 程序正常退出
- 使用System.exit()
- 终端使用Ctrl+C触发的中断
- 系统关闭
- 使用Kill pid命令干掉进程，使用kill -9 pid是不会JVM注册的钩子不会被调用。