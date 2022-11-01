---
layout: post
title: Terraform Provider如何解析.tr文件
category: IaC
catalog: true
published: true
tags:
  - IaC
time: '2022.10.31 20:54:00'
---
# 一、背景介绍
对于一个Terraform文件（.tf），相关providers是如何对资源`Resource`结构进行管理和资源映射应用的？如下方的k8s provider创建`kubernetes_cluster_role`的实例，是如何解析k8s资源信息？

```
resource "kubernetes_cluster_role" "example" {
  metadata {
    name = "terraform-example"
  }

  rule {
    api_groups = [""]
    resources  = ["namespaces", "pods"]
    verbs      = ["get", "list", "watch"]
  }
}
```

# 二、代码流走读
![]({{site.baseurl}}/img/20221031 k8s provider读取resource处理主要逻辑.png)
<img src="{{site.baseurl}}/img/20221031 k8s provider读取resource处理主要逻辑.png" width="100%"/>
# 三、参考文献
1. [kubernetes_cluster_role](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/cluster_role)
