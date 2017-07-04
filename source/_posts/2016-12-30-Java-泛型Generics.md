---
title: Java 泛型Generics
date: 2016-12-30 20:12:16
categories:
- Java SE
tags:
- 泛型
---

# 简介
Java泛型(Generics)是JDK 1.5中引入的一个新特性，允许在定义类和接口的时候使用类型参数（type parameter）。声明的类型参数在使用时用具体的类型来替换，泛型最主要的应用是在JDK 1.5中的新集合类框架中。

泛型使得编译器在编译时就可以发现很多错误，可以解决之前的集合类框架在使用过程中通常会出现的运行时类型错误。

# 类型擦除(type erasure)
类型擦除：使用泛型的时候加上类型参数，会在编译器进行编译的时候去掉。
> Java 语言中的泛型基本上完全在编译器中实现，由编译器执行类型检查和类型推断，然后生成普通的非泛型的字节码。

例如：代码中定义的`List<Object>`和`List<String>`在编译之后都会变成`List`。

## 类型擦除的过程
1. 找到用来替换类型参数的具体类
这个具体类一般是`Object`，如果指定了类型参数的上界，则使用这个上界。把代码中的类型参数都替换成具体的类，同时去掉出现的类型声明，即去掉`<>`的内容。比如，`T get()`方法声明就变成了`Object get()`，`List<String>`就变成了`List`。
2. 生成桥接方法（bridge method）
这是由于擦除了类型之后的类可能缺少某些必须的方法。
比如下面的`MyString`类，当类型信息被擦除之后，就变成了`class MyString implements Comparable`，但是这样累`MyString`就会有编译错误，因为没有实现接口`Comparable`声明的`int compareTo(Object)`方法。这个时候就由**编译器来动态生成**这个方法。
```java
class MyString implements Comparable<String>{
	public int compareTo(String str){
    	return 0;
    }
}
```

## 注意
1. 泛型类没有自己独有的Class类对象。
比如不存在`List<String>.class`或者`List<Integer>.class`，而只有`List.class`。经过类型擦除后只剩下原始类型。
2. 静态变量是被泛型类的所有实例所共享的。
对于声明为`MyClass<T>`的类，访问其中的静态变量的方法仍然是`MyClass.staticVar`。不论是通过`new MyClass<String>`还是`new MyClass<Integer>`创建的对象，都共享一个静态变量。
3. 泛型的类型参数不能用在异常处理的catch语句中。
异常处理是由JVM在运行时刻来进行的，异常类型被擦除后JVM将无法区分。

# 通配符与上下界
## 通配符
通配符代表的是一组类型，但是具体的类型是未知的。

`List<?>`所声明的就是List中包含的元素类型是未知的，但是`List<?>`并不等同于`List<Object>`。`List<Object>`实际上确定了List中包含的是Object及其子类，而`List<?>`其中所包含的元素是不确定的。

因为`List<?>`的中元素类型是未知的，所以不能通过`new ArrayList<?>`的方式来创建一个新的ArrayList对象（因为编译器无法知道具体的类型是什么）。
```java
public void test(List<?> list){
	list.add(1); // 编译错误
}
```

## 上下界
`List<?>` 中的元素类型是未知的，但是可以使用上下界来限制未知类型的范围。当引入了上界类之后，在使用类型的时候就可以使用上界类中定义的方法。

`List<? extends Number>` 说明List中可能包含的元素类型是Number及其子类。访问`List<? extends Number>`的时候，可以使用Number类的方法（比如`intValue()`等）。
`List<? super Number>`说明List中包含的元素类型是Number及其父类。

> Tips：当你需要从一个数据结构中获取数据时(get)，那么就使用 `? extends T`；如果你需要存储数据(put)到一个数据结构时，那么就使用 `? super T`； 如果你又想存储数据，又想获取数据，那么就不要使用通配符 `?` ，即直接使用具体泛型`T`。

```java
List<? extends Parent> exParentlist = new ArrayList<>();
exParentlist.add(new Child()); // compile error, 编译器无法检测安全性
exParentlist.add(new Parent()); // compile error
exParentlist.add(new GrandParent()); // compile error

List<? super Parent> superParentList = new ArrayList<>();
superParentList.add(new GrandParent()); // compile error
superParentList.add(new Parent()); // ok
superParentList.add(new Child()); // ok, Parent为下界，因此它的子类是安全的。
```

# 泛型的类型系统
引入泛型之后的类型系统增加了两个维度：一个是类型参数自身的继承体系结构，另外一个是泛型类或接口自身的继承体系结构。第一个指的是对于 `List<String>`和 `List<Object>`这样的情况，类型参数String是继承自Object的。而第二种指的是 List接口继承自Collection接口。对于这个类型系统，有如下的一些规则：
* **相同类型参数**的泛型类的关系取决于泛型类自身的继承体系结构。即`List<String>`是`Collection<String>` 的子类型，`List<String>`可以替换`Collection<String>`。这种情况也适用于带有上下界的类型声明。
* 当泛型类的类型声明中使用了**通配符**的时候， 其子类型可以在两个维度上分别展开。如对`Collection<? extends Number>`来说，其子类型可以在Collection这个维度上展开，即`List<? extends Number>`和`Set<? extends Number>`等；也可以在Number这个层次上展开，即`Collection<Double>`和 `Collection<Integer>`等。如此循环下去，`ArrayList<Long>`和 `HashSet<Double>`等也都算是`Collection<? extends Number>`的子类型。
* 如果泛型类中包含多个类型参数，则对于每个类型参数分别应用上面的规则。

# 自定义泛型类
一个类可以有多个类型参数，如 `MyClass<X, Y, Z>`。 每个类型参数在声明的时候可以指定**上界**（不能指定下界super）。
所声明的类型参数在Java类中可以像一般的类型一样作为方法的参数和返回值，或是作为域和局部变量的类型。
> 但是由于类型擦除机制，类型参数并**不能用来**创建对象或是作为静态变量的类型。

```java
class ClassTest<X extends Number, Y, Z> {
    private X x;
    private static Y y; //编译错误，不能用在静态变量中
    public X getFirst() {
        //正确用法
        return x;
    }
    public void wrong() {
        Z z = new Z(); //编译错误，不能创建对象
    }
}
```
# 自定义泛型方法
```java
/**
 * 一个简单的泛型方法
 */
public static <T> void out(T t) {
    System.out.println(t);
}
```
方法的参数彻底泛化了，这个过程涉及到编译器的**类型推导**和**自动打包**，也就说原来需要我们自己对类型进行的判断和处理，现在编译器帮我们做了。这样在定义方法的时候不必考虑以后到底需要处理哪些类型的参数，大大增加了编程的灵活性。

```java
/**
 * 泛型方法结合可变参数
 */
public static <T> void out(T... args) {
    for (T t : args) {
        System.out.println(t);
    }
}

public static void main(String[] args) {
    out("findingsea", 123, 11.11, true);
}
```

# 泛型参数的自动推断
```java
public class GenericUtil{
    public static <T> void out(T t) {
    	System.out.println(t);
    }
}
```
## 显示指定泛型
```java
GenericUtil.<String>out("aa");
```
## 隐式指定泛型
隐式指定泛型，其实就是让编译器自己去推断。可以根据接收的参数类型来推断出T，或者根据方法赋值的目标参数来推断。
```java
GenericUtil.out("aa");
```


* * *
感谢：
http://www.infoq.com/cn/articles/cf-java-generics