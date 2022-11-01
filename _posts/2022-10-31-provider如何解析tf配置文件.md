---
layout: post
title: Terraform Provider如何解析.tf文件
category: IaC
catalog: true
published: true
tags:
  - IaC
time: '2022.10.31 20:54:00'
---
# 一、背景介绍
对于一个Terraform文件（.tf），相关providers是如何对资源`Resource`结构进行管理、资源映射到代码和最终应用？如下方的[k8s provider](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/cluster_role)创建`kubernetes_cluster_role`的实例，是如何解析k8s资源信息？
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
## 2.1 主要逻辑走读
terraform provider的实现是通过调用[terraform-plugin-sdk](https://github.com/hashicorp/terraform-plugin-sdk)的[Provider](https://github.com/hashicorp/terraform-plugin-sdk/blob/b5b7dd0ab159303da4a64c94d64aeaea884c2a23/helper/schema/provider.go#L50)的[实例化](https://github.com/hashicorp/terraform-provider-kubernetes/blob/main/kubernetes/provider.go#L36)来实现。  
provider中最终要的一个环节实际是管理`Resource`资源，配置文件中所有被管理的资源对象均被抽象为`Resouce`资源。而`Resource`资源中的属性类型则被抽象为`Schema`结构体。下方代码就是`kubernetes cluster role`资源的定义逻辑。
```golang
kubernetes/resource_kubernetes_cluster_role.go

// k8s ClusterRole资源定义
func resourceKubernetesClusterRole() *schema.Resource {
    return &schema.Resource{
        CreateContext: resourceKubernetesClusterRoleCreate,
        ReadContext:   resourceKubernetesClusterRoleRead,
        UpdateContext: resourceKubernetesClusterRoleUpdate,
        DeleteContext: resourceKubernetesClusterRoleDelete,
        Importer: &schema.ResourceImporter{
            StateContext: schema.ImportStatePassthroughContext,
        },

        Schema: map[string]*schema.Schema{
            "metadata": metadataSchemaRBAC("clusterRole", false, false),
            "rule": {
                Type:        schema.TypeList,
                Description: "List of PolicyRules for this ClusterRole",
                Optional:    true,
                Computed:    true,
                MinItems:    1,
                Elem: &schema.Resource{
                    Schema: policyRuleSchema(),
                },
            },
        ......
}
```
k8s provider主要的代码执行流程图如下所示。
![]({{site.baseurl}}/img/20221031 k8s provider读取resource处理主要逻辑.png)
# 三、参考文献
1. [kubernetes_cluster_role](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/cluster_role)
