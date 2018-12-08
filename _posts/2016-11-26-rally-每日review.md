---
published: true
layout: post
title: rally项目发展每日自我review
category: openstack-infra
tags:
  - openstack-infra
time: '2016.11.26 23:58:00'
excerpt: 对rally项目社区进展进行每日review回顾。
---
对rally项目的每日发展进行自我review。
meeting wiki:https://wiki.openstack.org/wiki/Meetings/Rally
<!--more-->
## 社区rally项目自我review
| 链接             | 标题                | 重点内容                | 知会角色和对象|
|:----------------------:|:--------------------------:|:------------------------------:|:-------:|
|https://review.openstack.org/#/c/307415/| Make glance v2 the default| 原有的glance v1以及task yaml更新到默认|Andrey Kurilin|
|https://review.openstack.org/#/c/404886/| Unify exporters| 关于exporters这块内容，task和verify有代码冗余，进行了接口复用重构|None|
|https://review.openstack.org/#/c/411320| [verify][reports] Add comparison of verifications|verifications结果的比较| Andrey Kurilin自己提的patch|
|https://review.openstack.org/#/c/409781/1| [ci] Use neutron quotas for security groups| 将nova-net中安全组的测试迁移至neutron|nobody|
|https://review.openstack.org/#/c/408083/| [Core] Save atomic actions in new format|原子性操作的重构，但由于要向前兼容，老的原子性操作ACTIONS_SCHEMA还是继续保存一段时间|Alexander Maretskiy|
|https://review.openstack.org/#/c/390519/| Add nova.CreateFlavorAndAddTenantAccess scenario|创建一个flavor并使得一个tenant能有访问该tenant的权限|Andrey Kurilin;Chris St. Pierre;Konstantin Baikov;Roman Vasylets;Staroverov Anton|
|Replace six.iteritems with dict.items()|Replace six.iteritems with dict.items()|谈论了six.iteritems和dict.items()的性能效率。在一般没有大量数据的情况下，我们调用dict.iteritems就可以解决问题|Alberto Planas; Rajath Agasthya|
|https://review.openstack.org/#/c/404887/| Add junit exporter to verification| verify命令中添加一个叫export的子命令，该命令用于导出junit的测试报告,该功能方便rally和其他软件对接|no body|
|https://review.openstack.org/#/c/394356/| Add neutron lbaas v2 scenarios| 添加了一个lbaas的场景测试用例|Andrey Kurilin|
|https://review.openstack.org/#/c/395554/| Add Subtask and Workload classes|verify重构这块的model层替换|Andrey Kurilin;Alexander Maretskiy;Illina Khudoshyn|
|https://review.openstack.org/#/c/404168/| [Core] Subtask context|原有TaskEngine运行task的粒度是task，现在改为subtask|andrey kurilin以及Roman Vasylets|
|https://review.openstack.org/#/c/402102/|Additional checks for nova scenarios|nova场景用例在创建网络、安全组角色等动作时添加了判断操作（即/nova/utils.py对部分操作返回了操作结果）。因此设计到这些被改造的utils方法的一些场景用例添加了操作返回结果的判断操作|Andrey Kurilin;chenhb|
|https://review.openstack.org/#/c/393885/| [db] Migrate old verification results to new format|背景：现在社区正在对tempest的支持进行重构-->这需要对数据库也要进行重构，该patch的提交主要是对数据库架构的正确性进行验证，并且要求是兼容老的tempestDB数据库|Andrey Kurilin的自验过程，Yaroslav Lobankov review了该架构设计|
|https://review.openstack.org/#/c/297020/77| Task table changed to reflect new schema|数据表的变更|两个PTL以及Rally Group成员还有其他开发者|
|https://review.openstack.org/#/c/234195/ | Add ability to increase rps with certain duration|主要是考虑到cloud加载资源的性能是随着时间变化而变化的，因此引入了rps这个概念 |一个PTL以及若干个Reviewer|
