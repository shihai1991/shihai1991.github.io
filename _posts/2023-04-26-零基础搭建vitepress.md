---
layout: post
title: 零基础搭建vitepress总结
category: web
catalog: true
published: true
tags:
  - web
time: '2023.04.01 14:52:00'
---

> 需要用到搭建一个简易web网站管理一些markdown文件，约等于无前端基础。
> 搭建的环境是centos

# 一、基础环境安装
## 1.1 配置nodejs
```shell
// 下载nodejs包
wget https://nodejs.org/dist/v20.0.0/node-v20.0.0.tar.gz
tar zxvf node-v20.0.0.tar.gz
// 编译安装
cd node-v20.0.0
./configure --prefix=/usr/local/node/20.0.0
// 执行编译和安装。尴尬，没看到 https://nodejs.org/dist/v20.0.0/目录下有编译好的包，node-v20.0.0-linux-x64.tar.gz解压可以直接用
make & make install
```

## 1.2 配置yarn
`yarn` 是另外一个nodejs包管理器，可以理解是`npm`的替代品。
```
npm install -g yarn
```

# 二、vitepress搭建和运行

# 参考文献
[nodejs安装](https://www.runoob.com/nodejs/nodejs-install-setup.html)
[yarn简介](https://zhuanlan.zhihu.com/p/357454908)