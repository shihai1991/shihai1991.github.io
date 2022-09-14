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
# 查看系统配置信息
sysctl --system
```

## 1.2 通过yum源安装k8s
配置各类系统、docker-ce、k8s安装源。
```
# 配置系统基础的yum源
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

# 安装yum工具类
yum install -y yum-utils device-mapper-persistent-data lvm2 net-tools
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
安装相关组件。
```
# 安装docker-ce
yum -y install docker-ce
# 安装k8s相关组件（安装1.25.0版本在`kubeadm init`阶段各种失败，原因未知，先限定安装1.16.0，后面有时间在折腾）
yum install -y kubectl-1.16.0-0 kubeadm-1.16.0-0 kubelet-1.16.0-0
```
如果安装过程中提示`Peer's certificate issuer has been marked as not trusted by the user.`问题，请检查各个yum源中配置的baseurl是否是https协议，如果是https请修改为http协议即可。

## 1.3 配置容器相关代理
由于访问过程中需要到各类镜像中心拉取k8s组件镜像，但是国内因为网络问题导致下载受限，需要配置网络代理进行访问，当然使用国内镜像源也可以，请自行谷歌或本文参考文章。
```
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/https-proxy.conf
#在https-proxy中添加网络代理信息
[Service]
Environment="HTTPS_PROXY=http://user:pwd@proxy:port/"
Environment="NO_PROXY= localhost,127.0.0.1"
```
重启docker服务。
```
systemctl daemon-reload
systemctl restart docker
```

## 1.4 初始化k8s集群
初始化k8s集群，kubernetes-version可以通过`kubelet --version`进行查询
```
kubeadm init --apiserver-advertise-address=0.0.0.0 \
--apiserver-cert-extra-sans=127.0.0.1 \
--ignore-preflight-errors=all \
--kubernetes-version=v1.16.0 \
--service-cidr=10.10.0.0/16 \
--pod-network-cidr=10.18.0.0/16
```
如果在初始化阶段提示了`X509xx`问题。

## 1.5 安装网络驱动


# 二、相关组件介绍
pass

# 参考文档
1. [centos单机安装k8s](https://blog.51cto.com/u_15144750/3113358)
2. [CentOS 搭建 K8S，一次性成功，收藏了！](https://segmentfault.com/a/1190000037682150)