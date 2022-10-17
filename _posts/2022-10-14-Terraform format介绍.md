---
layout: post
title: Terraform format介绍
category: IaC
catalog: true
published: true
tags:
  - IaC
time: '2022.10.14 17:42:00'
---
# 一、背景介绍
Terraform使用文本文件描述基础设施和设置变量，这些文件被称为Terraform配置文件。Terraform配置文件主要有两种格式：Terraform格式(.tf)和JSON格式(.tf.json)。Terraform配置文件主要由provider、resource、data source和variable组成。

# 二、格式介绍

# 2.1 provider、resource
我们用[之前博客](https://shihai1991.github.io/iac/2022/06/10/Terraform%E5%88%9D%E4%BD%93%E9%AA%8C/)中介绍的一个示例来做分析。
```
terraform {
  # providers代表一个服务供应商，Terraform通过插件机制与Provider进行交互。
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.13.0"
    }
  }
}

# 对docker provider进行配置说明
provider "docker" {}

# docker_image是资源类型名，nginx是创建出的资源名
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  # docker_iamge.nginx.latest为引用资源的属性
  image = docker_image.nginx.latest
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}
```

# 2.2 variable


# 三、参考文献
1. [TERRAFORM TUTORIAL - TERRAFORM FORMAT(TF), INTERPOLATION(VARIABLES) & TERRAFORM CONSOLE](https://www.bogotobogo.com/DevOps/Terraform/Terraform-terraform-format-tf-and-interpolation-variables.php)  
2. [Terraform 基础知识](https://support.huaweicloud.com/basics-terraform/basics-terraform.pdf)
3. [Terraform variables](https://www.terraform.io/language/values/variables)