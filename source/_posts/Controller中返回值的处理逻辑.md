---
title: Controller中返回值的处理逻辑
date: 2023-09-07 15:07:10
categories:
  - 开发相关
tags: [Java, Spring]
---
<mark>建设中</mark>

在一次开发文件上传下载的任务中，学习了Spring中`Resource`类的使用, 不是`@Resource`, 本地测试项目是可以正常下载的，但是代码提交上去后发现返回的始终都是序列化的对象,而不是具体的文件, 通过对Spring返回值的学习和进一步追踪才找到问题所在。
# ResponseEntity\<T\> 类的使用