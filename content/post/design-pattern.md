---
title: "单例模式"
date: "2017-10-09 16:24:48"
tags: ["设计模式"]
categories: ["架构"]
---
单例模式Singleton
作用：保证整个应用程序中某个实例有且只有一个
类型：饿汉模式和懒汉模式


首先创建一个Pattern类，什么都不写。
```java
package top.richmanroad.demo;

public class Pattern {

}
```
再新建一个测试Test类
```java
package top.richmanroad.demo;

public class Test {
public static void main(String[] args) {
    
    Pattern pt1=new Pattern();
    Pattern pt2=new Pattern();
    Pattern pt3=new Pattern();
    System.out.println(pt1);
    System.out.println(pt2);
    System.out.println(pt3);
}
}
```
控制台输出结果
```java
top.richmanroad.demo.Pattern@15db9742
top.richmanroad.demo.Pattern@6d06d69c
top.richmanroad.demo.Pattern@7852e922
```
可以看出其三次输出的地址不相同，说明此时创建了三个不同的实例
## 饿汉模式
将构造方法私有化，不能被外部直接创建对象，此时在Test类中就不能直接实例化对象了，然后在Pattern类中创建唯一的实例，并且将其变为类的成员(static将方法变为类所有),这样在Test类中可以直接使用；类名.成员名来获取对象。此时Pattern类如下：
```java
package top.richmanroad.demo;

public class Pattern {
//将构造方法私有化，不能被外部直接创建对象
    private Pattern(){}
//创建类的唯一实例
    static Pattern instance=new Pattern();
}
```
Test类：
```java
package top.richmanroad.demo;

public class Test {
public static void main(String[] args) {
    
    Pattern pt1=Pattern.instance;
    Pattern pt2=Pattern.instance;
    Pattern pt3=Pattern.instance;
    System.out.println(pt1);
    System.out.println(pt2);
    System.out.println(pt3);
}
}
```
控制台输出结果：
```java
top.richmanroad.demo.Pattern@15db9742
top.richmanroad.demo.Pattern@15db9742
top.richmanroad.demo.Pattern@15db9742
```
可以看出三者的地址指向一处，说明此时只创建了一个实例。

接下来为了安全，需要将对Pattern类进行封装(面向对象思想)，此时我们的Pattern类就变成了如下内容
```java
package top.richmanroad.demo;

public class Pattern {
//将构造方法私有化，不能被外部直接创建对象
    private Pattern(){}
//创建类的唯一实例
 private static Pattern instance=new Pattern();
//提供一个用于获取实例的方法
 public static Pattern getInstance(){
     return instance;
 }
    
}
```
Test类修改内容并输出，结果同上
```java
Pattern pt1=Pattern.getInstance();
```
上面这种方法为饿汉模式，意思大致为:static为静态修饰符，当Pattern类加载时它就会创建一个唯一的实例，不管后面有没有调用，都随类一起产生。
## 懒汉模式
LazyPattern类
```java
package top.richmanroad.demo;

public class LazyPattern {
    //构造方法私有化
private LazyPattern(){
}
//声明类的唯一实例，使用private修饰
private static LazyPattern instance;
//提供一个用于获取实例的方法，使用public static修饰
public static LazyPattern getInstance() {
    if(instance==null){
        instance=new LazyPattern();
    }
    return instance;
}

}
```
类中只是声明了一个唯一的实例，并没有创建，只有当用户获取实例时才去判断这个实例是否为空

|    |饿汉模式|懒汉模式|
|:---:|:---:|:---:|
|区别 |加载时速度慢，运行时获取对象比较快|加载速度快，运行时获取对象速度比较慢|
|线程|安全|不安全|