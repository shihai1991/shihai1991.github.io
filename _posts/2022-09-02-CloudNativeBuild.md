---
layout: post
title: CloudNativeBuild使用及总结
category: iac
catalog: true
published: true
tags:
  - iac
time: '2022.09.02 10:36:00'
---
# 一、基本概念介绍
- buildpack：buildpack的职责是收集你的应用在`build`和`run`阶段所需的所有要素；
- builder：builder是一个包含执行`build`所需所有组件的镜像。

![buildpack](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/16/Buildpacks_img1.jpg)

# 二、安装及使用
下载示例代码。
```
git clone https://github.com/buildpacks/samples
```
`cd`到示例代码的一个应用。
```
cd samples/apps/java-maven
```
通过[pack](https://buildpacks.io/docs/tools/pack/)构建应用。
```
pack build myapp --builder cnbs/sample-builder:bionic
```
执行过程如下所示，由于我司有网络限制，具体的错误后面有时间在自己个人虚拟机上在执行试试。
![]({{site.baseurl}}/img/2022/Q3/2022090201 pack build.png)
