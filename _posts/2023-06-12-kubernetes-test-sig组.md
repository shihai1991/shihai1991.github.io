---
layout: post
title: kubernetes test SIG组梳理
category: k8s
catalog: true
published: true
tags:
  - k8s
  - test
time: '2023.06.12 09:35:00'
---
k8s测试基础设施的项目清单：
![]({{site.baseurl}}/img/2023/Q2/20230612-k8s-test-infra.png)

从PR触发测试执行流程：
![]({{site.baseurl}}/img/2023/Q2/20230612-test-infra工作流.png)

[k8s test-infra代码仓](https://github.com/kubernetes/test-infra)

# 一、自动化
## prow
**用途**：基于k8s的CI/CD系统。
主要的组件清单：
- 
- 
- 
![]({{site.baseurl}}/img/2023/Q2/20230612-prow-architecture.png)

[Prow Status](https://prow.k8s.io/)
补充prow status图

## kubetest1/2

## boskos
存在价值：
架构设计：

## bootstrap
存在价值：
架构设计：

## config

# 二、可视化
# 三、其他
## kind

# 四、参考文献
- [k8s testing sig](https://github.com/kubernetes/community/blob/master/sig-testing/README.md)
