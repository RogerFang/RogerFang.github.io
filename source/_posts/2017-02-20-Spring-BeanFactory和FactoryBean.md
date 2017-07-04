---
title: 'Spring: BeanFactory和FactoryBean'
date: 2017-02-20 19:10:50
categories:
- Spring
---

# BeanFactory
BeanFactory 定义了IoC容器应该遵守的最底层和最基本的编程规范。所有的Bean都是由BeanFactory来进行管理的。

# FactoryBean
FactoryBean 就是一个Bean，只不过这是一个**能产生或修饰对象生成**的工厂Bean。是为了方便Factory实例在Spring IoC中的配置而产生的。
```java FactoryBean
public interface FactoryBean<T> {
	T getObject() throws Exception;
	Class<?> getObjectType();
	boolean isSingleton();
}
```

一般情况下，Spring通过反射机制利用<bean>的class属性指定实现类来实例化Bean。在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息。配置的方式的灵活性是受限的，这时采用编写代码的方式可能会得到一个简单的方案。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
FooFactoryBean ffb=context.getBean("&fooFactoryBean");
```

BeanFactory的属性`String FACTORY_BEAN_PREFIX = "&";`专门为 FactoryBean 设置的，FactoryBean的实现类默认返回的对象是`getObject()`的类型，如果我们就要返回FactoryBean的实现本身的实例就要用到这个前缀&了。