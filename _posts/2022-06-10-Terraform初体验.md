---
layout: post
title: Terraform初体验
category: IaC
catalog: true
published: true
tags:
  - IaC
time: '2022.06.10 10:49'
---

# 一、Infrastructure as Code(IaC)
## 1.1. IaC起源
IaC确切来源已无法考证，实际还有个概念可以理解成IaC框架的扩展：配置即代码（CaC）。你可以将CaC理解成IaC的一种理念扩展，主要的CaC工具如下图所示，最早的CaC工具是1993年开发的CEFngine。

| Tool                  | Released by          | Method        |
| --------------------- | -------------------- | ------------- |
| Chef                  | Chef (2009)          | Pull          |
| Otter                 | Inedo (2015)         | Push          |
| Puppet                | Puppet (2005)        | Pull          |
| SaltStack             | SaltStack (2011)     | Push and Pull |
| CFEngine              | Northern.tech (1993) | Pull          |
| Terraform             | HashiCorp (2014)     | Push          |
| Ansible/Ansible Tower | Red Hat (2012)       | Push          |

## 1.2. IaC目标
维基定义：基础架构即代码 (IaC) 是通过机器可读的定义文件而不是物理硬件配置或交互式配置工具来管理和配置计算机数据中心的过程。

# 二、Terraform初次使用
## 2.1. 在centos7上安装
配置hashicorp的yum源并安装。
```shell
yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
yum install terraform -y
```
安装完成后查看terraform的版本。
```shell
terraform version
```

## 2.2. 使用
本文尝试用terraform来控制docker管理相关容器实例，先创建一个测试目录。
```
mkdir learn-terraform-docker-container
cd        
```
将下面的内容拷贝进main.tf文件中。
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
在`learn-terraform-docker-container`目录中执行`terraform init`，执行这个动作会下载一个用于terraform和docker进行交互的插件。  
如果遇到了如下超时问题，则可以参考此[issue](https://github.com/hashicorp/terraform/issues/27742)解决。
```
# root @ jz2e in ~/learn-terraform-docker-container [16:02:22]
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 2.13.0"...
╷
│ Error: Failed to install provider
│
│ Error while installing kreuzwerker/docker v2.13.0: could not query provider registry for registry.terraform.io/kreuzwerker/docker: failed to retrieve
│ authentication checksums for provider: the request failed after 2 attempts, please try again later: Get
│ "https://github.com/kreuzwerker/terraform-provider-docker/releases/download/v2.13.0/terraform-provider-docker_2.13.0_SHA256SUMS": net/http: request
│ canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
``` 
最后执行`terraform apply`并输入`yes`则就可以通过terraform创建出一个nginx容器。如果你想删除这个容器，则可以执行`terraform destroy`来执行。
```
# root @ jz2e in ~/learn-terraform-docker-container [16:16:53]
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                      NAMES
225a4c834b1b   0e901e68141f   "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds   0.0.0.0:8000->80/tcp       tutorial
```

# 三、引用附录
-[History of Infra as Code](https://www.infoq.com/presentations/history-infra-as-code/)  
-[Infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code)  
-[tefform官网教程](https://learn.hashicorp.com/tutorials/terraform/infrastructure-as-code)  
-[terraform providers](https://registry.terraform.io/browse/providers)  
-[Terraform 实战 | 万字长文](https://posts.careerengine.us/p/6254c3bc8407c2569699ad83)
