---
title: "面向对象的理解"
date: "2019-10-08 12:12:20"
tags: ["面向对象"]
categories: ["架构"]
---

# 背景

说起面向对象的特征，大多数人肯会不加思索的答封装、继承、多态，可能还有抽象。最近在学 DDD 的时候，感觉它更加合理的运用了设计模式和面向对象的思想去解决复杂的业务场景，还有是阅读源码的时候也会看到大量的设计模式，所以说 Design Pattern 和 Object-Oriented 熟练的话会提升很大的代码质量（编程范式后续还会讲函数式和响应式）。

下面只说自己的体会和见解，如有异议欢迎留言。



# 四大特征

## 抽象

抽象，隐藏与上下文无关的信息，抽象的目的是为了简单，从宏观的角度去处理问题，举例：你开车去上班，但你只要知道车能带你到公司而不用在乎车内部的复杂零件是如何做工的，此时车就抽象成为一个整体。

## 封装

将功能相近的代码放到同一个类中，这样做的好处是能够进行解耦让不同的代码各司其职，具体细分为 **Information hiding** 和 **Implementation hiding** 

```java
/**
* 通过 访问修饰符 讲一些细节保留在当前类中，而一些相对外开放的则使用 public 进行修饰
*
*/
class InformationHiding 
{
    //Restrict direct access to inward data
    private ArrayList items = new ArrayList();
 
    //Provide a way to access data - internal logic can safely be changed in future
    public ArrayList getItems(){
        return items;
    }
}
```

```java
interface ImplemenatationHiding {
    Integer sumAllItems(ArrayList items);
}
class InformationHiding implements ImplemenatationHiding
{
    //Restrict direct access to inward data
    private ArrayList items = new ArrayList();
 
    //Provide a way to access data - internal logic can safely be changed in future
    public ArrayList getItems(){
        return items;
    }
 
    public Integer sumAllItems(ArrayList items) {
        //Here you may do N number of things in any sequence
        //Which you do not want your clients to know
        //You can change the sequence or even whole logic
        //without affecting the client
    }
}
```



## 继承

继承就是子类继承父类并可以父类或者重写父类的方法，这样做能让基础功能代码复用，但是现在有一种说法是组合优先于继承，因为继承也有自身的局限性，容易把代码写死。



## 多态

这里的个人理解就是一段代码写在不同的业务场景下有不同的功能，通过重载或者重写，其正规的说法是：

> In java language, polymorphism is essentially considered into two versions:
>
> - Compile time polymorphism (static binding or **method overloading**)
> - Runtime polymorphism (dynamic binding or **method overriding**)



