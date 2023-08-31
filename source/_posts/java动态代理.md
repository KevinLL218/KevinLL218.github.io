---
title: java动态代理
date: "2023-08-31 10:29:38"
update: "2023-08-31 17:21:22"
categories:
  - 开发相关
tags: [Java, Spring]
---

<!-- TOC -->
* [前言](#前言-)
* [事务](#事务)
* [失效](#失效)
* [动态代理](#动态代理)
  * [Spring AOP 和 AspectJ](#spring-aop-和-aspectj)
    * [AspectJ](#aspectj)
    * [Spring AOP](#spring-aop)
  * [JDK Proxy 和 Cglib](#jdk-proxy-和-cglib)
* [总结](#总结)
<!-- TOC -->

# 前言 

某个同事写了逆天代码，大概情况如下

```java
interface Demo {
    String selectByConditionSharding(String name);
}
```

逆天在哪呢，根据条件查，但是条件是固定的，名字带分片，实际一点关系都没有. 为别人阅读代码提供困难，这是个人代码习惯的问题了，不再赘述.

但是在后面的代码中，看到这两行让我有所沉思

```java
class Demo {
    @Autowired
    ApplicationContext context;

    public void doSth() {
        Object bean = context.getBean("BeanName");
        bean.doSomeThing();
    }
}
```

看的出来,他考虑到了**事务失效**的情况,但是这让我陷入思考:  
1. 哪些情况下事务会失效
2. 同一个类中`this`调用一定会失效吗 (不一定)
3. Spring AOP 和 AspectJ 和 cglib到底怎么回事

带着以上问题，我去谷歌一下，最终整理了一下.

# 事务

# 失效

# 动态代理

## Spring AOP 和 AspectJ
> 这两个东西常常放在一起说明，实际上是两个截然不同的东西。

现在常用的Spring AOP实际上**借用了**AspectJ的一些注解.致使我认为SpringAOP和AspectJ是同一个东西, 在Spring配置中开启`<aop:aspectj-autoproxy/>`即可识别AspectJ的一些注解

### AspectJ
> AspectJ其实(在某些意义下)应该和Lombok并列的存在，某种程度独立于JVM，在编译期间就对类进行织入，实际上运行时已经是被代理的类了。也就是说使用AspectJ代理的类，在加载时就不是它自己了。

因为它是在{编译期间, 编译之后, 加载前}进行织入，所以运行期间就没有它什么事了。

### Spring AOP
> SpringAOP则是对AOP概念在Spring架构下的一种实现，只在Spring容器中才能实现，无法脱离容器，而且也是在运行期间进行代理。

由于SpringAOP是运行期间进行动态代理的，所以在同一个类中调用自己的方法，是不会被代理，所以使用`@Transactional`主键的类事务就失效了.

如果让类内调用保留事务，则必须使用AspectJ来实现动态代理。可以在配置中切换为AspectJ代理

```xml
<tx:annotation-driven  mode="aspectj"/>
```

## JDK Proxy 和 Cglib
> 动态代理有两种实现方式，一种是JDK原生的方法，另一种则是使用字节码技术。

JDK原生方法实际上就是通过代码实现`代理模式`, 具体实现方式网上有很多了。总之就是必须要实现接口，编写代码。  
而使用Cglib则比原生JDK方式更自由，不需要显式实现接口，而是在编译期对源代码进行改写。

都不能对final修饰的类和方法进行代理。

# 总结