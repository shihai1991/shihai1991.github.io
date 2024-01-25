---
layout: post
title: 在docker上运行nova-compute
category: openstack
catalog: true
published: false
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
遇到的一些坑（下面有些涉及密码仅涉及此开源项目配置，无其他秘钥）：
#### 1. keystone
a. 直接用镜像拉keystone实例，容器镜像中会缺少MySQL-python，需要手动安装MySQL-python或者通过docker直接构建运行；  
b. 非admin用户鉴权无法通过，提示401问题，需要在keystone.conf配置文件中的[DEFAULT]中添加default_domain_Id=default的id；

#### 2. glance
a. glance的镜像中没有安装openstack客户端，只有keystone的客户端，但此客户端版本太低，是0.10.1，所以需要在keystone实例里面创建一些资源信息，/etc/hosts需要自己配置一下否则连不上其他节点服务；  
```shell
openstack --debug user create --domain default --password GLANCE_PASS glance
openstack role add --user glance --project service admin
openstack service create --name glance image --description "OpenStack Image Service"
openstack endpoint create  --region regionOne image public http://glance:9292
openstack endpoint create  --region regionOne image internal http://glance:9292
openstack endpoint create  --region regionOne image admin http://glance:9292
```
b. i18n/_message.py中抛UnicodeError错误，是一个调用直接抛错的逻辑，需要把/usr/lib/python2.7/dist-packages/oslo/i18n/_message.py 167行的__str__()函数直接注释掉；

#### 3. nova-controller
a. 和glance类似，只安装了一个keystone低版本，需要到keystone节点实例里面连接
```shell
openstack user create --domain default --password NOVA_PASS nova
openstack role add --user nova --project service admin
openstack service create --name nova compute --description "OpenStack Compute"
openstack endpoint create --region regionOne compute public http://controller:8774/v2/%\(tenant_id\)s
openstack endpoint create --region regionOne compute internal http://controller:8774/v2/%\(tenant_id\)s
openstack endpoint create --region regionOne compute admin http://controller:8774/v2/%\(tenant_id\)s
```
# 模型定义
## 通过模型定义测试依赖
TBD

# 参考文章
- [docker-nova-compute](https://github.com/int32bit/docker-nova-compute?tab=readme-ov-file)
- [Nova的作用 openstack openstack nova架构](https://blog.51cto.com/u_13354/8102414)
- [Identity API V2 or V3](https://docs.openstack.org/keystone/10.0.3/http-api.html)
- [Identity API V2 docs](https://sergslipushenko.github.io/html/api-ref-identity-admin-v2.html)
- [OpenStack Nova 总结（01）- 架构及组件详解](https://blog.csdn.net/dylloveyou/article/details/80698420)
- [OpenStack虚拟机创建流程](https://blog.csdn.net/dylloveyou/article/details/78587308)
- [Setting Openstack compute node with a fake hypervisor](https://stackoverflow.com/questions/67677398/setting-openstack-compute-node-with-a-fake-hypervisor)
