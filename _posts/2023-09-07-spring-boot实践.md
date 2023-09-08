---
layout: post
title: spring-boot学习实践
category: spring
catalog: true
published: false
tags:
  - spring
time: '2023.09.07 20:53:00'
---

# FAQ
- org.springframework.boot:spring-boot-starter-parent:pom:3.0.0 failed to transfer from xxx
在setting中的Maven.importing.VM options for importer中填入：-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true    

# 参考文章
- [Spring Boot开发](https://www.liaoxuefeng.com/wiki/1252599548343744/1266265175882464)