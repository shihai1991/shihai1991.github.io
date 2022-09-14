---
layout: post
title: centos7上单机安装k8s
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

# 修改内核参数
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

## 1.2 通过yum源安装k8s
```
# 配置系统基础的yum源
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

# 配置docker-ce安装源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 配置k8s安装源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

# 参考文档
1. [centos单机安装k8s](https://blog.51cto.com/u_15144750/3113358)
2. [CentOS 搭建 K8S，一次性成功，收藏了！](https://segmentfault.com/a/1190000037682150)