---
title: java动态代理
date: "2023-08-31 10:29:38"
categories:
  - 开发相关
tags: [Java, Spring]
---



# 

某个同事写了逆天代码，大概意思是

```java
interface Demo {
    String selectByConditionSharding(String name);
}
```

逆天在哪呢，根据条件查，但是条件是固定的，名字带分片，实际一点关系都没有. 为别人阅读代码提供困难

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

看的出来
