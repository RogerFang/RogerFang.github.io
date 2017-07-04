---
title: 'Mybatis: Mapper映射文件解析'
date: 2017-02-19 15:25:06
categories:
- Mybatis
---

解析Mybatis的配置文件最后一步是解析mapper元素，也就是解析其对应的Mapper.xml文件。

Mapper元素只有一个属性**namespace**，它有两个作用：
* 一是用于区分不同的mapper（在不同的mapper文件里，子元素的id可以相同，mybatis通过namespace和子元素的id联合区分）；
* 二是与接口关联，生成Mapper代理对象。

Mapper配置文件的顶级子元素：
* **cache**：配置本地命名空间的缓存。
* **cache-ref**：从其他命名空间引用缓存配置。
* **resultMap**：用来描述如何从数据库结果集中来加载对象。
* **parameterMap**（废弃）
* **sql**：可被其他语句引用的可重用语句块。
* **insert|update|delete|select**：数据库操作的4种statement。

`MappedStatement`对象封装了**statement**元素的所有属性以及子节点值，`MappedStatement`对象有一个id属性用于唯一标记它，这个id由namespace加statement元素的id属性值构成。创建好的`MappedStatement`对象存入`Configuration`对象的mappedStatements缓存中，key为MappedStatement对象的id值。

具体参见：[Mapper XML映射文件](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html)

