---
layout: post
title: Terraform初体验 
category: IaC 
catalog: true
published: False
tags:
  - IaC 
time: '2022.06.10 10:49'
---

# 一、Infrastructure as Code(IaC)
## IaC起源
IaC确切来源无法考证，实际还有个概念和IaC戚戚相关：配置即代码（CaC）。你可以将CaC理解成IaC的一种扩展。
| Tool | Released by |Method | Approach | Written in | Comments |
| ---- | ---- | ---- | ---- | ---- | ---- |
|Chef|Chef (2009)|Pull|Declarative and imperative|Ruby|-|
|Otter|Inedo|Push|Declarative and imperative|-|Windows-oriented|
|Puppet|Puppet (2005)|Pull|Declarative and imperative|C++ & Clojure since 4.0, Ruby|-|
|SaltStack|SaltStack (2011)|Push and Pull|Declarative and imperative|Python|-|
|CFEngine|Northern.tech|Pull|Declarative|C|-|
|Terraform|HashiCorp (2014)|Push|Declarative|Go|-|
|Ansible / Ansible Tower|Red Hat (2012)|Push|Declarative and imperative|Python|-|

## IaC目标
维基定义：基础架构即代码 (IaC) 是通过机器可读的定义文件而不是物理硬件配置或交互式配置工具来管理和配置计算机数据中心的过程。

# 二、Terraform

# 三、引用附录
-[History of Infra as Code](https://www.infoq.com/presentations/history-infra-as-code/)
