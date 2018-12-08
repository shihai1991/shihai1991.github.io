---
published: true
layout: post
title: openstack社区测试相关工具调研
category: openstack
tags:
  - openstack
time: '2017.07.17 00:00:00'
excerpt: os社区测试相关工具调研总结
---
openstack社区测试相关工具调研总结。目前最活跃的项目是tempest、rally。

<!--more-->

## os社区

### 1. openstack/dox

  * **活跃度**：年patch不超过10个，基本死亡;

  * **主要介绍**：dox项目的灵感来源于tox和python的virtualenv。该工具使用docker容器运行本地测试;
其核心功能围绕：
    
    1）什么命令需要被运行？
    
    2）需要采用何种image？

### 2. openstack/browbeat

  * **活跃度**：活跃

  * **主要介绍**：browbeat是os社区的性能调优和分析工具。其主要功能有：运行perfkit、rally、shaker及自动化安装部署数据分析工具（其本质为对各类工具的再次封装）。
  
  * **命令行执行方式**：`browbeat.py <workload> #perfkit、rally、shaker or all`
  
  * **REF**:
  
    [browbeat程序主入口](https://github.com/openstack/browbeat/blob/master/browbeat.py)
    
    [browbeat对其他工具的支撑入口](https://github.com/openstack/browbeat/tree/master/lib)

### 3. openstack/cloudcafe
  
  * **活跃度**：不活跃
  
  * **主要介绍**：基于OpenCAFE引擎的OpenStack自动化测试框架。
  
  * **openstack/opencafe**：开源通用自动化框架引擎，该框架被设计用来支持所有的测试方法论（如：单元测试、功能测试及集成测试），主要指令：`cafe-config plugins list`。
  
  * **openstack/cloudroast**：基于CloudCafe自动化测试套件。
  
  * **点评**：该项目的设计理念与rally的设计理念几乎相同，但是该项目的生命周期没有被更好的维护（eg:没有完备的文档支持）。

### 4. openstack/performa

  * **活跃度**：2016年开始的项目，贡献者只有创始人一个人（shakhat，也是shaker的创始人，目前在华为北美研究所）。

  * **主要介绍**：分布式场景测试，用于场景测试、结果处理以及报告生成。

### 5. openstack/shaker
  
  * **活跃度**：不活跃
  
  * **主要介绍**：shaker封装了主流的网络测试工具（如：iperf、iperf3以及netperf）。shaker通过heat定义的模板来创建OpenStack实例和网络并执行相关的测试工具。
  
  * **核心架构**：shaker工具由服务器和代理模式组成。服务器由shaker命令执行，负责部署实例，执行方案中制定的测试及结果处理和报告生成。shaker-agent是轻量级别的框架并且以伦寻的方式server端得到任务并反馈最后执行结果。
  ![shaker's architecture]({{site.baseurl}}/img/shaker_architecture.svg)
  
  
  
  * **该项目产生背景**：缺少rally团队的支持而产生。

### 6. openstack/tempest

  * **活跃度**:活跃
  
  * **基本介绍**：tempest是OpenStack集成测试套件，主要有api测试、场景测试。
  
### 7. openstack/rally

  * **活跃度**：活跃
  
  * **基本介绍**:rally是openstack社区的性能测试框架，插件式架构使得rally可以进行其他项目的测试（如：docker、k8s）。
  
  * **REF**:
      [rally弹药库（非openstack相关测试插件）](https://github.com/xrally)

### 8. openstack/performance-docs

  * **基本介绍**：社区的性能结果归档地址，由OpenStack Performance Work-Group进行维护。
  
## 其他
1. 谷歌云开源项目[GoogleCloudPlatform](https://github.com/GoogleCloudPlatform)
