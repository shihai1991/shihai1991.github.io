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
## 2.1 初始化项目
```shell
# 创建一个目录作为vitepress项目的根目录
mkdir vitepress-starter && cd vitepress-starter
# 初始化项目，会生成一个完整的vitepress示例项目
yarn vitepress init
```
用`yarn vitepress init`初始化项目需要通过终端配置一些配置项。
![](https://github.com/vuejs/vitepress/blob/main/docs/guide/vitepress-init.png)

初始化后的目录结构如下图所示：
```
$ tree -a -I node_modules
.
├── docs
│   ├── api-examples.md
│   ├── index.md
│   ├── markdown-examples.md
│   └── .vitepress
│       ├── cache
│       │   ├── deps
│       │   │   ├── _metadata.json
│       │   │   ├── package.json
│       │   │   ├── @theme_index.js
│       │   │   ├── @theme_index.js.map
│       │   │   ├── vue.js
│       │   │   └── vue.js.map
│       │   └── deps_temp_316bd56c
│       │       ├── package.json
│       │       ├── vue.js
│       │       └── vue.js.map
│       └── config.ts
├── package.json
└── yarn.lock

5 directories, 15 files
```

## 2.2 执行项目
如果用`yarn vitepress init`生成初始化项目，那就可以跳过此步骤。否则就需要在package.json文件添加如下的npm执行脚本用来运行项目：
```
{
  ...
  "scripts": {
    "docs:dev": "vitepress dev docs",
    "docs:build": "vitepress build docs",
    "docs:preview": "vitepress preview docs"
  },
  ...
}
```
执行如下命令启动一个服务：
```
//或者 npm run docs:dev
yarn docs:dev --host 0.0.0.0
```

## 2.3 项目扩展
TODO，可以参考[vitepress的官方指导](https://github.com/vuejs/vitepress/tree/main/docs/guide)。

# 三、FAQ

### Q1: 执行yarn遇到：unable to verify the first certificate.
可以配置`yarn`取消ssl校验，配置命令：`yarn config set "strict-ssl" false -g`。

### Q2：已经有了一个项目怎么安装依赖软件包？
通过在项目路径下执行`yarn install`来安装就可以了。`yarn install`安装会索引`package.json`文件中的`dependencies`依赖软件包。

# 四、参考文献
[nodejs安装](https://www.runoob.com/nodejs/nodejs-install-setup.html)
[yarn简介](https://zhuanlan.zhihu.com/p/357454908)
[VitePress 手把手完全使用手册](https://juejin.cn/post/7164276166084263972#heading-8)
[vitepress 入门](https://github.com/vuejs/vitepress/blob/main/docs/guide/getting-started.md)