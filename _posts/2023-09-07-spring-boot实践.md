---
layout: post
title: spring-boot学习实践
category: spring
catalog: true
published: true
tags:
  - spring
  - java
time: '2023.09.07 20:53:00'
---
> 学习中，先把能在搜索引擎上找到的信息搬砖搬到一起。

# [Spring WebFlux](https://docs.spring.io/spring-framework/docs/5.0.0.M5/spring-framework-reference/html/web-reactive.html)
Spring框架中的响应式堆栈框架。

## [Reactive Programming](https://spring.io/blog/2016/06/07/notes-on-reactive-programming-part-i-the-reactive-landscape)
响应式编程是关于异步和事件驱动的非阻塞式应用。

## [Reactor](https://projectreactor.io/)
Spring框架在内部使用`Reactor`来提供自身的反应式支持。

![](https://docs.spring.io/spring-framework/docs/5.0.0.M5/spring-framework-reference/html/images/webflux-overview.png)

# 集成[OpenAPI](https://www.openapis.org/)
spring有两种方式可以支持OpenAPI swagger UI。实际动作就是添加或者更新依赖。
一种是支持Spring WebMvc。
```maven
   <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-starter-webmvc-api</artifactId>
      <version>2.2.0</version>
   </dependency>
```
而另外一种则是支持Spring WebFlux。[详情](https://springdoc.org/modules.html)
```maven
   <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-starter-webflux-ui</artifactId>
      <version>2.2.0</version>
   </dependency>
```

`springdoc-openapi-starter-xxx-ui`都是`springdoc-openapi-v2`系列的软件包，而`springdoc-openapi-v1`的软件包则略有不同。[详情](https://springdoc.org/#migrating-from-springdoc-v1)

# FAQ
- org.springframework.boot:spring-boot-starter-parent:pom:3.0.0 failed to transfer from xxx
需要判断三个点：
1. jdk版本是否符合配置要求
2. 在setting中的Maven.Runner.VM Options中填入：-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true
3. 在setting中的Maven.importing.VM options for importer中填入：-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true

- Cannot resolve symbol 'var'
确保IDE内的所有Language Level配置都是按项目jdk版本要求进行配置。

# 参考文章
- [Spring Boot开发](https://www.liaoxuefeng.com/wiki/1252599548343744/1266265175882464)
