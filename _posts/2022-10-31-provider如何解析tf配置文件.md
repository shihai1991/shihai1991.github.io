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
对于一个HCL编写的TF文件，相关providers是如何对资源结构进行管理和资源映射应用的？如下方的`kubernetes_cluster_role`所创建的实例是如何调用k8s创建资源？

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

# 二、基本介绍
# 三、参考文献
1. [kubernetes_cluster_role](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/cluster_role)
