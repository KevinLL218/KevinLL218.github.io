---
title: Controller中返回值的处理逻辑
date: 2023-09-07 15:07:10
updated: 2024-04-18 22:30:00
categories:
  - 开发相关
tags: [Java, Spring]
toc: true
---

在一次开发文件上传下载的任务中，学习了Spring中`Resource`类的使用, 不是`@Resource`, 本地测试项目是可以正常下载的，但是代码提交上去后发现返回的始终都是序列化的对象,而不是具体的文件, 通过对Spring返回值的学习和进一步追踪才找到问题所在。

<!-- more -->

# @ResponseBody 注解的处理  

> `@ResponseBody`注解顾名思义，就是将Controller的返回值经过处理写入到 `Response`中的`Body`字段上.具体处理方式则是通过各种`Converter`的实现类结合`MediaType` 去处理转换。  

 虽然不知道谁控制着`Controller`注解，但是可以知道`@ResponseBody`注解是由`RequestResponseBodyMethodProcessor` 方法去处理的，该类同时还会处理方法参数中的`@RequestBody`注解,同样的，参数的读取也是通过"MessageConverter"来实现.

因此，理论上也是可以通过修改Converter来控制返回值的类型的。而这个脑瘫项目就是这么处理的，将所有的返回类型都一刀切,全部都是json返回类型。

# ContentNegotiationManagerFactoryBean
内容协商管理器工厂，就是控制入参和出参格式的类。这个工厂类是通过配置文件生成一个协商管理器，供Spring容器使用。可以通过配置文件配置支持的媒体类型和默认类型，而这个脑瘫项目就是只配置了json，默认也是`application/json`, 所以所有的返回值都以json字符串格式进行处理.  

值得注意的是，该类本身是由工厂模式实现，而构建协商管理器的过程则是策略模式。最终的协商过程也是策略模式。

# ResponseEntity\<T\> 类的使用

由于内容协商的修改，所以在本项目中,ResponseEntity已经失去了他的作用。  

传统的MVC对于下载的处理方式是比较臃肿的，需要打开`HttpServletResponse`的输出流，设置响应头属性，和MIME属性，而这些操作既然都是一样的，那么`dont't repeat yourself` , 这也是该类的使用场景。  
对于文件流，可以直接作为返回值，由内容协商管理器自动进行判断实现文件下载。

```java
@RestController
class Demo {
    @GetMapping("file")
    public ResponseEntity<Resource> download() {
        return new ResponseEntity<>(resource, HttpStatus.OK);
    }
}
```

还是由于项目中，修改了内容协商管理器，导致所有返回类型都是json，所以即使是文件流，其返回到前端依旧是json格式.

# RequestMappingHandlerAdapter  
通过查阅资料及翻阅源码，大概知道了为什么自定义了`MessageConverter`导致ResponseEntity失效了。  
顾名思义,`RequestMappingHandlerAdapter`就是处理`@RequestMapping`的类，接着会发现在这个类中有个字段是 `HandlerMethodReturnValueHandlerComposite`, 这是一个组合模式的实现类，处理方法返回值，通过debug得知运行时自动添加返回值处理策略，这些策略或者以`Handler`结尾，或者以`Processor`结尾，最终都会添加到这个组合类中,运行时根据返回值的类型进行动态地处理，比如以`ResponseEntity`作为返回值，会由`HttpEntityMethodProcessor`类进行处理。  
程序到了这里， `ResponseEntity`还是能够正常处理的，但是当真正写结果的时候，发现是通过其父类的方法实现，而父类的方法则是上文所述的`MediaType`及其`HttpMessageConverter`实现类，原本根据不同的`MediaType`使用对应的`Converter`，此时由于只定义了`json`格式，其他格式也都默认为json，以至于最终的所有返回结果都是`JSONString`了。

所以说，这个💩山项目自作聪明的方法，断绝了后续开发的扩展性，除非将其大改，否则就是💩上雕花。