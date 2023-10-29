---
layout: post
title: springboot加载配置文件源码剖析
category: java
catalog: true
header-style: text
published: false
tags:
  - java
  - spring
time: '2023.10.28 16:55:00'
---

# 代码走读
我用的spring-boot版本是2.7.15。在`spring-boot/src/main/resources/META-INF/spring.factories`中定义了各类配置工厂方法。  
而配置源文件的相关加载器配置如下：
```
# PropertySource Loaders
org.springframework.boot.env.PropertySourceLoader=\
org.springframework.boot.env.PropertiesPropertySourceLoader,\
org.springframework.boot.env.YamlPropertySourceLoader
```
代码走读的最快最直接的方式是断点调试，当我们在`YamlPropertySourceLoader`中设置断点并执行后我们就可以获取到完整的调用链：
```
load:46, YamlPropertySourceLoader (org.springframework.boot.env)
load:54, StandardConfigDataLoader (org.springframework.boot.context.config)
load:36, StandardConfigDataLoader (org.springframework.boot.context.config)
load:108, ConfigDataLoaders (org.springframework.boot.context.config)
load:132, ConfigDataImporter (org.springframework.boot.context.config)
resolveAndLoad:87, ConfigDataImporter (org.springframework.boot.context.config)
withProcessedImports:116, ConfigDataEnvironmentContributors (org.springframework.boot.context.config)
processInitial:240, ConfigDataEnvironment (org.springframework.boot.context.config)
processAndApply:227, ConfigDataEnvironment (org.springframework.boot.context.config)
postProcessEnvironment:102, ConfigDataEnvironmentPostProcessor (org.springframework.boot.context.config)
postProcessEnvironment:94, ConfigDataEnvironmentPostProcessor (org.springframework.boot.context.config)
onApplicationEnvironmentPreparedEvent:102, EnvironmentPostProcessorApplicationListener (org.springframework.boot.env)
onApplicationEvent:87, EnvironmentPostProcessorApplicationListener (org.springframework.boot.env)
doInvokeListener:176, SimpleApplicationEventMulticaster (org.springframework.context.event)
invokeListener:169, SimpleApplicationEventMulticaster (org.springframework.context.event)
multicastEvent:143, SimpleApplicationEventMulticaster (org.springframework.context.event)
multicastEvent:131, SimpleApplicationEventMulticaster (org.springframework.context.event)
environmentPrepared:85, EventPublishingRunListener (org.springframework.boot.context.event)
lambda$environmentPrepared$2:66, SpringApplicationRunListeners (org.springframework.boot)
accept:-1, 1922930974 (org.springframework.boot.SpringApplicationRunListeners$$Lambda$73)
forEach:1259, ArrayList (java.util)
doWithListeners:120, SpringApplicationRunListeners (org.springframework.boot)
doWithListeners:114, SpringApplicationRunListeners (org.springframework.boot)
environmentPrepared:65, SpringApplicationRunListeners (org.springframework.boot)
prepareEnvironment:343, SpringApplication (org.springframework.boot)
run:301, SpringApplication (org.springframework.boot)
run:1303, SpringApplication (org.springframework.boot)
run:1292, SpringApplication (org.springframework.boot)
main:31, Application (com.xxxx.xxxx)
```

# 参考文档
- [Stackoverflow: In a spring boot project, how to load application.yaml into Java Properties](https://stackoverflow.com/questions/43796664/in-a-spring-boot-project-how-to-load-application-yaml-into-java-properties)
- [Stackoverflow: In which class in the source code of spring-boot or spring is the application.yml or application.properties file processed?](https://stackoverflow.com/questions/73352379/in-which-class-in-the-source-code-of-spring-boot-or-spring-is-the-application-ym)
