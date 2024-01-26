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

# openstack服务
openstack的核心服务有：keystone、nova、neutron、cinder和glance。
![OpenStack Nova架构 侵权删]({{site.baseurl}}/img/2024/Q1/20240115-nova组件调用.png)

## keystone
TBD

## nova
- nova api: 接收用户的API请求，来源于用户的API请求或者用户界面的操作；
- nova scheduler: 虚拟机调度服务，负责决定在哪个计算节点上运行虚拟机；
- nova conductor: `nova compute`访问数据库的能力交给了`nova conductor`，确保当`nova compute`节点被攻击劫持后系统数据的安全；
- nova compute: 管理虚拟机的核心服务，通过调用虚拟机管理程序`Hypervisor API`实现虚拟机的生命周期管理；

### nova-compute
`nova-compute`组件的[main函数](https://github.com/openstack/nova/blob/6531ed6310c4c11ee807e10d1a0fa91658c3afea/nova/cmd/compute.py#L44)在`nova/cmd/compute.py`文件中。
```python
def main():
    ...
    server = service.Service.create(binary='nova-compute',
                                    topic=compute_rpcapi.RPC_TOPIC)
    service.serve(server)
    service.wait()
```

当我们启动`nova-compute`组件时，`WSGIService`类就会根据`main函数`中指定的`binary='nova-compute')`查找倒`nova-compute`的管理器`ComputeManager`并启动。
```python
# 定义了各种的管理器manager
SERVICE_MANAGERS = {
    'nova-compute': 'nova.compute.manager.ComputeManager',
    'nova-conductor': 'nova.conductor.manager.ConductorManager',
    'nova-scheduler': 'nova.scheduler.manager.SchedulerManager',
}

...
class WSGIService(service.Service):
    ...
    # 获取相关管理器
    def _get_manager(self):
        """Initialize a Manager object appropriate for this service.

        Use the service name to look up a Manager subclass from the
        configuration and initialize an instance. If no class name
        is configured, just return None.

        :returns: a Manager instance, or None.

        """
        manager = SERVICE_MANAGERS.get(self.binary)
        if manager is None:
            return None

        manager_class = importutils.import_class(manager)
        return manager_class()
...
```
`ComputeManager`负责和`Hypervisor`交互的核心功能，所有的虚拟化计算驱动driver都在[`nova/virt`目录](https://github.com/openstack/nova/tree/master/nova/virt)中。
![nova compute组件类图]({{site.baseurl}}/img/2024/Q1/20240126-nova-compute组件类图.png)
通过对nova-compute.conf文件中的`compute_driver`配置进行变更就可以实现对接不同的虚拟化平台。
```shell
[DEFAULT]
compute_driver=libvirt.LibvirtDriver
[libvirt]
virt_type=kvm
```
在所有的虚拟层驱动中，有个驱动driver是专门用来做测试的，这个驱动名字是`FakeDriver`，在[文件`nova/virt/fake.py`中](https://opendev.org/openstack/nova/src/branch/master/nova/virt/fake.py)。这个驱动用于控制平面服务的性能测试或者不同计算节点间的“移动”操作。`FakeDriver`驱动不会和任何的虚拟机管理平台`Hypervisor`进行通信，也不会和周边的neutron、cinder、glance服务通信。

## neutron
TBD

## cinder
TBD

## glance
TBD

# 安装
## 安装基础工具

### 安装docker
TBD
### 安装docker-compose
TBD

## 安装openstack
我使用的是[docker-nova-compute](https://github.com/int32bit/docker-nova-compute)在docker容器中安装openstack，因为这个项目已经是10年的项目并且没有更新，使用过程会遇到一些坑：
#### 1. 安装基础中间件服务
执行以下命令从docker镜像源拉取rabbitmq、mariadb镜像并运行：
```shell
docker run -d -e RABBITMQ_NODENAME=rabbitmq -h rabbitmq --name rabbitmq rabbitmq:latest
docker run -d -e MYSQL_ROOT_PASSWORD=MYSQL_DBPASS -h mysql --name mysql -d mariadb:latest
```

#### 2. 安装[keystone](https://github.com/int32bit/docker-keystone)
执行以下命令，运行keystone服务：
```shell
docker run -d  --link mysql:mysql --name keystone -h keystone krystism/openstack-keystone:latest
```
运行过程会遇到两个问题，需要我们手动规避解决：
- 直接用镜像拉keystone实例，容器镜像中会缺少MySQL-python，需要手动安装MySQL-python或者通过docker直接构建运行；  
- 非admin用户鉴权无法通过，提示401问题，需要在keystone.conf配置文件中的[DEFAULT]中添加default_domain_Id=default的id；

#### 3. 安装[glance](https://github.com/int32bit/docker-glance)
执行以下命令，运行glance服务：

运行过程会遇到两个问题，需要我们手动规避解决：
- glance的镜像中没有安装openstack客户端，只有keystone的客户端，但此客户端版本太低，是0.10.1，所以需要在keystone实例里面创建好资源信息，如下命令行所示。在`/etc/hosts`中需要配置一下节点和ip信息，否则无法连不上其他节点服务；  
```shell
openstack --debug user create --domain default --password GLANCE_PASS glance
openstack role add --user glance --project service admin
openstack service create --name glance image --description "OpenStack Image Service"
openstack endpoint create  --region regionOne image public http://glance:9292
openstack endpoint create  --region regionOne image internal http://glance:9292
openstack endpoint create  --region regionOne image admin http://glance:9292
```
- `i18n/_message.py`中抛`UnicodeError`错误，是一个调用直接抛错的逻辑，需要把`/usr/lib/python2.7/dist-packages/oslo/i18n/_message.py`中的第167行的`__str__()`函数直接注释掉；

#### 4. 安装nova-controller
执行以下命令，运行nova-controller服务：
```shell
docker run -d --link mysql:mysql --link keystone:keystone --link rabbitmq:rabbitmq --link glance:glance \
-e OS_USERNAME=admin \
-e OS_PASSWORD=ADMIN_PASS \
-e OS_AUTH_URL=http://keystone:5000/v2.0 \
-e OS_TENANT_NAME=admin \
--privileged \
--name controller \
-h controller \
krystism/openstack-nova-controller:latest
```
运行过程会遇到和glance类似的问题，镜像中就只安装了一个keystone低版本，需要到keystone节点实例里面创建好相关资源信息。
```shell
openstack user create --domain default --password NOVA_PASS nova
openstack role add --user nova --project service admin
openstack service create --name nova compute --description "OpenStack Compute"
openstack endpoint create --region regionOne compute public http://controller:8774/v2/%\(tenant_id\)s
openstack endpoint create --region regionOne compute internal http://controller:8774/v2/%\(tenant_id\)s
openstack endpoint create --region regionOne compute admin http://controller:8774/v2/%\(tenant_id\)s
```

#### 5. 安装nova-compute
执行以下命令，运行nova-compute服务：
```shell
docker run -d --link mysql:mysql --link keystone:keystone --link rabbitmq:rabbitmq --link glance:glance --link controller:controller \
-e OS_USERNAME=admin \
-e OS_PASSWORD=ADMIN_PASS \
-e OS_AUTH_URL=http://keystone:5000/v2.0 \
-e OS_TENANT_NAME=admin \
--privileged \
--name node1 \
-h node1 \
krystism/openstack-nova-compute:latest
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
- [Nova hypervisors](https://github.com/openstack/nova/blob/master/doc/source/admin/configuration/hypervisors.rst)
- [devstack: Fake virt driver](https://docs.openstack.org/devstack/latest/guides/nova.html#fake-virt-driver)
- [nova与neutron交互的细节分析](https://www.jianshu.com/p/fc8ecdf11ec8)
