---
title: IDEA 搭建 Tomcat9 源码调试环境
date: 2017-01-20 15:01:42
categories:
- tomcat
tags:
- tomcat
---

# 准备
1. IDEA
2. Tomcat源码：apache-tomcat-9.0.0.M17-src.zip 版本
3. Maven环境
4. JDK 1.8

# 环境搭建
1. 解压tomcat源码
2. 在解压根目录下添加`pom.xml`文件，转换为Maven工程
	```xml
            <?xml version="1.0" encoding="UTF-8"?>
        <project xmlns="http://maven.apache.org/POM/4.0.0"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

            <modelVersion>4.0.0</modelVersion>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>Tomcat9.0</artifactId>
            <name>Tomcat9.0</name>
            <version>9.0</version>

            <build>
                <finalName>Tomcat9.0</finalName>
                <sourceDirectory>java</sourceDirectory>
                <testSourceDirectory>test</testSourceDirectory>
                <resources>
                    <resource>
                        <directory>java</directory>
                    </resource>
                </resources>
                <testResources>
                    <testResource>
                        <directory>test</directory>
                    </testResource>
                </testResources>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-compiler-plugin</artifactId>
                        <version>3.6.0</version>

                        <configuration>
                            <encoding>UTF-8</encoding>
                            <source>1.8</source>
                            <target>1.8</target>
                        </configuration>
                    </plugin>
                </plugins>
            </build>

            <dependencies>
                <dependency>
                    <groupId>junit</groupId>
                    <artifactId>junit</artifactId>
                    <version>4.12</version>
                    <scope>test</scope>
                </dependency>
                <dependency>
                    <groupId>org.easymock</groupId>
                    <artifactId>easymock</artifactId>
                    <version>3.4</version>
                </dependency>
                <dependency>
                    <groupId>ant</groupId>
                    <artifactId>ant</artifactId>
                    <version>1.7.0</version>
                </dependency>
                <dependency>
                    <groupId>wsdl4j</groupId>
                    <artifactId>wsdl4j</artifactId>
                    <version>1.6.2</version>
                </dependency>
                <dependency>
                    <groupId>javax.xml</groupId>
                    <artifactId>jaxrpc</artifactId>
                    <version>1.1</version>
                </dependency>
                <dependency>
                    <groupId>org.eclipse.jdt.core.compiler</groupId>
                    <artifactId>ecj</artifactId>
                    <version>4.5.1</version>
                </dependency>
            </dependencies>
        </project>
    ```
3. 在解压根目录下新建`catalina-home`目录，作为tomcat的运行目录
4. 将`conf`、`webapps`目录移到`catalina-home`目录下
5. 在IDEA中打开该maven项目
6. 设置启动
	6.1 Run > Edit configurations... > Application
	6.2 配置`main class`：`org.apache.catalina.startup.Bootstrap`。
    6.3 配置`vm options`：`-Dcatalina.home="D:\Workspace\java\source\apache-tomcat-9.0.0.M17-src\catalina-home"`。
7. 运行启动

# 问题
1. 启动时出现如下异常
	解决办法：删除`webapps`目录下的`examples`目录
    ```java
    严重: Error configuring application listener of class listeners.ContextListener
    java.lang.ClassNotFoundException: listeners.ContextListener
        at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1275)
        at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1109)
        at org.apache.catalina.core.DefaultInstanceManager.loadClass(DefaultInstanceManager.java:520)
        at org.apache.catalina.core.DefaultInstanceManager.loadClassMaybePrivileged(DefaultInstanceManager.java:501)
        at org.apache.catalina.core.DefaultInstanceManager.newInstance(DefaultInstanceManager.java:118)
        at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4639)
        at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5179)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
        at org.apache.catalina.core.ContainerBase.addChildInternal(ContainerBase.java:752)
        at org.apache.catalina.core.ContainerBase.addChild(ContainerBase.java:728)
        at org.apache.catalina.core.StandardHost.addChild(StandardHost.java:734)
        at org.apache.catalina.startup.HostConfig.deployDirectory(HostConfig.java:1107)
        at org.apache.catalina.startup.HostConfig$DeployDirectory.run(HostConfig.java:1841)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)

    一月 20, 2017 4:39:46 下午 org.apache.catalina.core.StandardContext listenerStart
    严重: Error configuring application listener of class listeners.SessionListener
    java.lang.ClassNotFoundException: listeners.SessionListener
        at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1275)
        at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1109)
        at org.apache.catalina.core.DefaultInstanceManager.loadClass(DefaultInstanceManager.java:520)
        at org.apache.catalina.core.DefaultInstanceManager.loadClassMaybePrivileged(DefaultInstanceManager.java:501)
        at org.apache.catalina.core.DefaultInstanceManager.newInstance(DefaultInstanceManager.java:118)
        at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4639)
        at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5179)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
        at org.apache.catalina.core.ContainerBase.addChildInternal(ContainerBase.java:752)
        at org.apache.catalina.core.ContainerBase.addChild(ContainerBase.java:728)
        at org.apache.catalina.core.StandardHost.addChild(StandardHost.java:734)
        at org.apache.catalina.startup.HostConfig.deployDirectory(HostConfig.java:1107)
        at org.apache.catalina.startup.HostConfig$DeployDirectory.run(HostConfig.java:1841)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)

    一月 20, 2017 4:39:46 下午 org.apache.catalina.core.StandardContext listenerStart
    严重: Error configuring application listener of class websocket.drawboard.DrawboardContextListener
    java.lang.ClassNotFoundException: websocket.drawboard.DrawboardContextListener
        at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1275)
        at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1109)
        at org.apache.catalina.core.DefaultInstanceManager.loadClass(DefaultInstanceManager.java:520)
        at org.apache.catalina.core.DefaultInstanceManager.loadClassMaybePrivileged(DefaultInstanceManager.java:501)
        at org.apache.catalina.core.DefaultInstanceManager.newInstance(DefaultInstanceManager.java:118)
        at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4639)
        at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5179)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
        at org.apache.catalina.core.ContainerBase.addChildInternal(ContainerBase.java:752)
        at org.apache.catalina.core.ContainerBase.addChild(ContainerBase.java:728)
        at org.apache.catalina.core.StandardHost.addChild(StandardHost.java:734)
        at org.apache.catalina.startup.HostConfig.deployDirectory(HostConfig.java:1107)
        at org.apache.catalina.startup.HostConfig$DeployDirectory.run(HostConfig.java:1841)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)
    ```
2. 访问`localhost:8080`出现500
	解决办法：从tomcat二进制版本下拷贝`lib`目录到本项目中的`catalina-home`目录下。
	```java
	java.lang.NullPointerException
	at org.apache.jsp.index_jsp._jspService(index_jsp.java:427)
	at org.apache.jasper.runtime.HttpJspBase.service(HttpJspBase.java:70)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
	at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:443)
	at org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:385)
	at org.apache.jasper.servlet.JspServlet.service(JspServlet.java:329)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:230)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:165)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:199)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:140)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
	at org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:624)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:87)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:349)
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:495)
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66)
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:767)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1347)
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:745)
    ```
