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
Terraform使用文本文件描述基础设施和设置变量，这些文件被称为Terraform配置文件。Terraform配置文件主要有两种格式：Terraform格式(.tf)和JSON格式(.tf.json)。

# 二、格式介绍
我们用[之前博客](https://shihai1991.github.io/iac/2022/06/10/Terraform%E5%88%9D%E4%BD%93%E9%AA%8C/)中介绍的一个示例来做分析。

```
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.13.0"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}
```

# 三、参考文献
1. [TERRAFORM TUTORIAL - TERRAFORM FORMAT(TF), INTERPOLATION(VARIABLES) & TERRAFORM CONSOLE](https://www.bogotobogo.com/DevOps/Terraform/Terraform-terraform-format-tf-and-interpolation-variables.php)
