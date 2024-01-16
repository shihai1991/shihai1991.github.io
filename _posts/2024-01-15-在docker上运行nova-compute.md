---
layout: post
title: 在docker上运行nova-compute
category: openstack
catalog: true
published: true
tags:
  - openstack
  - docker
  - test
time: '2024.01.15 16:17:00'
---
> 测试左移需要通过和物理环境解耦的来实现，否则仅测试用例的测试左移，但没有和物理环境解耦，环境的成本开销会随着测试的持续左移而变大（可以等价理解为用物理资源换测试效率）。

# openstack组件
![OpenStack Nova架构 侵权删]({{site.baseurl}}/img/2024/Q1/20240115-nova组件调用.png)
# 安装
## 安装基础工具
### 安装docker

## 安装openstack
### 安装/运行基础组件
### 安装/运行openstack

# 模型定义
## 通过模型定义测试依赖
TBD

# 参考文章
- [docker-nova-compute](https://github.com/int32bit/docker-nova-compute?tab=readme-ov-file)
- [Nova的作用 openstack openstack nova架构](https://blog.51cto.com/u_13354/8102414)
- [Identity API V2 or V3](https://docs.openstack.org/keystone/10.0.3/http-api.html)
