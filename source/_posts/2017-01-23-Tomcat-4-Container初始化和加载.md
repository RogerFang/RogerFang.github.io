---
title: 'Tomcat(4): Container初始化和加载'
date: 2017-01-23 10:09:31
categories:
- tomcat
---

Container分成4个级别的容器：Engine、Host、Context、Wrapper，它们是父子关系。

![](/images/tomcat/tomcat-container.png)

# 初始化
Container容器的初始化入口在`org.apache.catalina.startup.Catalina`类的`load()`方法中，其中调用了`Digester digester = createStartDigester();`得到Digester对象（其中设置了许多的解析规则），之后调用`digester.parse()`对`conf/server.xml`进行解析生成对象及其相互关系。
> Digeter是apache的common项目，是对 SAX 的封装，作用是将 XML 转化成对象。

```java Catalina.createStartDigester()
/**
 * 用于设置解析时用到的规则
 */
protected Digester createStartDigester() {
    ……

    // 如果遇到”Server“元素起始符;则创建"org.apache.catalina.core.StandardServer"的一个实例对象，并压入堆栈;如果"Server"元素的"className"属性存在，那么用这个属性的值所指定的class来创建实例对象，并压入堆栈。
    digester.addObjectCreate("Server",
                             "org.apache.catalina.core.StandardServer",
                             "className");

    // 从server.xml读取"Server"元素的所有{属性:值}配对,用对应的Setter方法将属性值设置到堆栈顶层元素(Server)。
    digester.addSetProperties("Server");

    // 遇到"Server"结束符时，调用“次顶层元素(Catalina)”的"setServer"方法，设置对象间的关系。
    digester.addSetNext("Server",
                        "setServer",
                        "org.apache.catalina.Server");
    ……
    //遇到标签使用规则
    digester.addRuleSet(new HostRuleSet("Server/Service/Engine/"));
    ……
}
```

digester对`conf/server.xml`设置的标签动作有5种调用：
* addObjectCreate：遇到起始标签的元素，初始化一个实例对象入栈
* addSetProperties：遇到某个属性名，使用setter来赋值
* addSetNext：遇到结束标签的元素，调用相应的方法
* addRule：调用rule的begin 、body、end、finish方法来解析xml，入栈和出栈给对象赋值
* addRuleSet：调用addRuleInstances来解析xml标签

## Server、Service解析
`conf/server.xml`中，Calatina的Server对象是`StandardServer`，Service对象是`StandardService`。以下代码来自`Catalina.createStartDigester()`方法，涉及Server和Service部分并省略了部分子节点。
```java Catalina.createStartDigester()
// Server
digester.addObjectCreate("Server",
                         "org.apache.catalina.core.StandardServer",
                         "className");
digester.addSetProperties("Server");
digester.addSetNext("Server",
                    "setServer",
                    "org.apache.catalina.Server");

// Service
digester.addObjectCreate("Server/Service",
                         "org.apache.catalina.core.StandardService",
                         "className");
digester.addSetProperties("Server/Service");
digester.addSetNext("Server/Service",
                    "addService",
                    "org.apache.catalina.Service");
// Service:Executor
digester.addObjectCreate("Server/Service/Executor",
                 "org.apache.catalina.core.StandardThreadExecutor",
                 "className");
digester.addSetProperties("Server/Service/Executor");

digester.addSetNext("Server/Service/Executor",
                    "addExecutor",
                    "org.apache.catalina.Executor");

// Service:Connector
digester.addRule("Server/Service/Connector",
                 new ConnectorCreateRule());
digester.addRule("Server/Service/Connector", new SetAllPropertiesRule(
        new String[]{"executor", "sslImplementationName", "protocol"}));
digester.addSetNext("Server/Service/Connector",
                    "addConnector",
                    "org.apache.catalina.connector.Connector");
```

## Engine、Host、Context解析
以下代码来自`Catalina.createStartDigester()`方法。其中`EngineRuleSet`、`HostRuleSet`、`ContextRuleSet`都定义了相应的解析规则。
```java
// Engine
digester.addRuleSet(new EngineRuleSet("Server/Service/"));
// Host
digester.addRuleSet(new HostRuleSet("Server/Service/Engine/"));
// Context
digester.addRuleSet(new ContextRuleSet("Server/Service/Engine/Host/"));
```

# Context容器加载web服务与热部署
Tomcat 通过BackgroundProcessor来实现Context的加载web服务和热部署：Tomcat 的 Engine 会启动一个线程，该线程每10s会发送一个事件，监听到该事件的部署配置类会自动去扫描 webapp 文件夹下的war包，将其加载成一个Context，即启动一个web服务。

##开启线程，执行BackGroupProcessor
1. `StandardEngine`的`startInternal()`方法调用父类`ContainerBase`的`startInternal()`方法，其中调用`threadStart()`方法开启一个线程。

    ```java ContainerBase.threadStart()
    /**
     * Start the background thread that will periodically check for
     * session timeouts.
     */
    protected void threadStart() {

        if (thread != null)
            return;
        if (backgroundProcessorDelay <= 0)
            return;

        threadDone = false;
        String threadName = "ContainerBackgroundProcessor[" + toString() + "]";
        thread = new Thread(new ContainerBackgroundProcessor(), threadName);
        thread.setDaemon(true);
        thread.start();

    }
    ```

2.该线程每10s会调用一次processChildren。

    ```java ContainerBase.ContainerBackgroundProcessor
    /**
     * Private thread class to invoke the backgroundProcess method
     * of this container and its children after a fixed delay.
     */
    protected class ContainerBackgroundProcessor implements Runnable {

        @Override
        public void run() {
            Throwable t = null;
            String unexpectedDeathMessage = sm.getString(
                    "containerBase.backgroundProcess.unexpectedThreadDeath",
                    Thread.currentThread().getName());
            try {
                while (!threadDone) {
                    try {
                        // 在StandardEngine中构造方法设置默认backgroundProcessorDelay=10，即10s调用一次
                        Thread.sleep(backgroundProcessorDelay * 1000L);
                    } catch (InterruptedException e) {
                        // Ignore
                    }
                    if (!threadDone) {
                        processChildren(ContainerBase.this);
                    }
                }
            } catch (RuntimeException|Error e) {
                t = e;
                throw e;
            } finally {
                if (!threadDone) {
                    log.error(unexpectedDeathMessage, t);
                }
            }
        }
        ...
    }
    ```

3.在`processChildren()`方法中又调用其子组件Engine、Host、Context、Wrapper及其相关组件的`backgroundProcess()`方法。

    `fireLifecycleEvent()`：对容器的监听对象发送Lifecycle.PERIODIC_EVENT事件，调用LifecycleListener的lifecycleEvent。

    ```java ContainerBase.backgroundProcess()
    /**
     * Execute a periodic task, such as reloading, etc. This method will be
     * invoked inside the classloading context of this container.
     */
    @Override
    public void backgroundProcess() {

        ...
        fireLifecycleEvent(Lifecycle.PERIODIC_EVENT, null);
    }
    ```

    ```java StandardContext.backgroundProcess()
    @Override
    public void backgroundProcess() {

        if (!getState().isAvailable())
            return;

        Loader loader = getLoader();
        if (loader != null) {
            try {
                // WebappLoader
                loader.backgroundProcess();
            } catch (Exception e) {
                log.warn(sm.getString(
                        "standardContext.backgroundProcess.loader", loader), e);
            }
        }
        Manager manager = getManager();
        if (manager != null) {
            try {
                manager.backgroundProcess();
            } catch (Exception e) {
                log.warn(sm.getString(
                        "standardContext.backgroundProcess.manager", manager),
                        e);
            }
        }
        WebResourceRoot resources = getResources();
        if (resources != null) {
            try {
                resources.backgroundProcess();
            } catch (Exception e) {
                log.warn(sm.getString(
                        "standardContext.backgroundProcess.resources",
                        resources), e);
            }
        }
        super.backgroundProcess();
    }
    ```

4.调用 WebappLoader 的`backgroundProcess()`方法，`reloadable`即为是否开启热部署，而`modified()`则是当前文件是否有修改的判断，当开启了热部署且有修改就会调用Context的reload方法进行重加载，实现web服务的**热部署**。
```java WebappLoader.backgroundProcess()
@Override
public void backgroundProcess() {
    if (reloadable && modified()) {
        try {
            Thread.currentThread().setContextClassLoader
                (WebappLoader.class.getClassLoader());
            if (context != null) {
                context.reload();
            }
        } finally {
            if (context != null && context.getLoader() != null) {
                Thread.currentThread().setContextClassLoader
                    (context.getLoader().getClassLoader());
            }
        }
    }
}
```

## 加载Web服务
在上述BackgroundProcessor的第3步中，调用`fireLifecycleEvent()`方法对容器的监听对象发送`Lifecycle.PERIODIC_EVENT`事件。

而在Digester**解析**规则定义阶段，Host的解析规则通过添加`HostRuleSet`完成，该类在对`StandardHost`解析时会增加一个`HostConfig`监听器。

`HostConfig`对该事件的响应方法`lifecycleEvent()`中调用其`check()`方法，在`check()`方法中会调用其`deployApps()`方法完成 Web Application的部署。
> 部署的过程，其实就是创建了Context对象，并添加到Host中。

```java HostConfig
@Override
public void lifecycleEvent(LifecycleEvent event) {
	...

    // Process the event that has occurred
    // 事件对应的处理
    if (event.getType().equals(Lifecycle.PERIODIC_EVENT)) {
        check();
    } else if (event.getType().equals(Lifecycle.BEFORE_START_EVENT)) {
        beforeStart();
    } else if (event.getType().equals(Lifecycle.START_EVENT)) {
        start();
    } else if (event.getType().equals(Lifecycle.STOP_EVENT)) {
        stop();
    }
}

/**
 * Check status of all webapps.
 */
protected void check() {

    if (host.getAutoDeploy()) {
        // Check for resources modification to trigger redeployment
        DeployedApplication[] apps =
            deployed.values().toArray(new DeployedApplication[0]);
        for (int i = 0; i < apps.length; i++) {
            if (!isServiced(apps[i].name))
                checkResources(apps[i], false);
        }

        // Check for old versions of applications that can now be undeployed
        if (host.getUndeployOldVersions()) {
            checkUndeploy();
        }

        // Hotdeploy applications
        deployApps();
    }
}

/**
 * Deploy applications for any directories or WAR files that are found
 * in our "application root" directory.
 */
protected void deployApps() {

    File appBase = host.getAppBaseFile();
    File configBase = host.getConfigBaseFile();
    String[] filteredAppPaths = filterAppPaths(appBase.list());
    // Deploy XML descriptors from configBase
    deployDescriptors(configBase, configBase.list());
    // Deploy WARs
    deployWARs(appBase, filteredAppPaths);
    // Deploy expanded folders
    deployDirectories(appBase, filteredAppPaths);

}
```

WebApp由三种**部署方式**：
* 在server.xml的Host标签中声明Context标签
* 将war包放入webapps中
* context.xml配置方式

**部署的过程**：就是通过任务框架`ExecutorService`执行一个具体的部署任务（这些任务类作为`HostConfig`的内部类）。每个部署任务执行过程都有下面一块代码完成Context对象的实例化并为其添加一个`ContextConfig`监听器，以及将其添加到对应的Host中。
```java
// StandardHost中configClass默认为"org.apache.catalina.startup.ContextConfig"
Class<?> clazz = Class.forName(host.getConfigClass());
LifecycleListener listener =
    (LifecycleListener) clazz.newInstance();
context.addLifecycleListener(listener);

host.addChild(context);
```

# Wrapper加载
在上面的 Context 部署的过程中有一步是给 Context 添加`ContextConfig`监听器。
而在 StandardContext 的`startInternal()`方法中，发送了监听事件`CONFIGURE_START_EVENT`。

```java StandardContext.startInternal()
// Notify our interested LifecycleListeners
fireLifecycleEvent(Lifecycle.CONFIGURE_START_EVENT, null);
```

`ContextConfig`监听到该事件，调用`configureStart()`方法，在该方法中调用`webConfig()`完成web.xml解析，生成servlet、filter等信息，并配置加载Wrapper。

> Wrapper 代表一个 Servlet，它负责管理一个 Servlet，包括的 Servlet 的装载、初始化、执行以及资源回收。


* * *
感谢：
阿里技术专家，楚岩：[Tomcat源码分析系列博客](https://yq.aliyun.com/tags/type_blog-tagid_2702/?spm=5176.8091938.0.0.FiZvcG)