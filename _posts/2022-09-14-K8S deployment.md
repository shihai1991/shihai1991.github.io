---
layout: post
title: k8s单机安装部署
category: k8s
catalog: true
published: true
tags:
  - k8s
time: '2022.09.14 15:15:00'
---
# 一、部署
## 1.1 关闭环境相关配置
```
# 关闭交换区
sudo swapoff -a #临时关闭 
sudo sed -i 's/.*swap.*/#&/' /etc/fstab #永久关闭交换区

# 禁用selinux
setenforce 0 #临时关闭
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config #永久关闭

# 关闭防火墙
sudo systemctl stop firewalld.service
sudo systemctl disable firewalld.service
```

# 参考文档
1. [centos单机安装k8s](https://blog.51cto.com/u_15144750/3113358)
2. [CentOS 搭建 K8S，一次性成功，收藏了！](https://segmentfault.com/a/1190000037682150)