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
Cloud Native Buildpacks可以将你的应用源码转换为在任意云上运行的容器镜像。
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
或者到buildpack路径中进行`build`构建。
```
cd buildpacks/java-maven
pack build sample-java-maven-app --builder cnbs/sample-builder:bionic --buildpack . --path ../../apps/java-maven
```
执行过程如下所示，由于我司有网络限制，具体的错误后面有时间在自己个人虚拟机上再执行试试。
![]({{site.baseurl}}/img/2022/Q3/2022090201 pack build.png)

# 三、生命周期
从上面的执行图中除了拉取`cnbs/sample-builder`构建容器外，你会发现还有`ANALYZING`、`DETECTING`、`RESTORING`、`BUILDING`和`BUILDING`过程。这其实是云原生应用的完整的生命周期，[详情](https://github.com/buildpacks/lifecycle)。
## 3.1 Build
- analyzer：从上一个应用镜像中读取`metadata`信息；
- detector：通过`/bin/detect`选择`buildpacks`并且生成一个`build`计划；
- restorer：从上一个应用镜像和缓存中恢复`layer metadata`及`cached layers`；
- builder：通过`/bin/build`执行`buildpacks`；
- exporter：创建镜像和`caches layers`。  
或者
- creator：运行上面的五个阶段，[代码执行详情](https://github.com/buildpacks/lifecycle/blob/4f52db0a90b5637e718328e7019bf8e592fdeece/cmd/lifecycle/creator.go#L162) [命令行参数传入creator](https://github.com/buildpacks/lifecycle/blob/4f52db0a90b5637e718328e7019bf8e592fdeece/cmd/lifecycle/creator.go#L54)。

## 3.2 Run
- launcher：调用目标进程。

## 3.3 Rebase
- rebaser：从上一个更新基础`layers`的image上创建image。

# 四、FAQ
- Q1：samples/buildpacks中的[build](https://github.com/buildpacks/samples/blob/main/buildpacks/java-maven/bin/build)、detect脚本在哪个代码里面被调用？  
A1：最开始的pack版本模块没有拆分，调用代码在pack工具中，现在最新版本的执行代码在[lifecycle](https://github.com/buildpacks/lifecycle/blob/4f52db0a90b5637e718328e7019bf8e592fdeece/buildpack/build.go#L101)中。

# 五、参考文档
[Creating container images with Cloud Native Buildpacks using AWS CodeBuild and AWS CodePipeline](https://aws.amazon.com/blogs/containers/creating-container-images-with-cloud-native-buildpacks-using-aws-codebuild-and-aws-codepipeline/)
[buildpack: build an app](https://buildpacks.io/docs/app-developer-guide/build-an-app/)
