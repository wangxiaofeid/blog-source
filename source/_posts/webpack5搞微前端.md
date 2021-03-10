---
title: webpack5搞微前端
date: 2021-03-10 16:50:20
tags: webpack5 微前端
---

本示例将演示如何利用 webpack5 的联邦模块支持项目微前端开发

## Module Federation（联邦模块）

### What

Module Federation 使 JavaScript 应用得以从另一个 JavaScript 应用中动态地加载代码 —— 同时共享依赖。如果某应用所消费的 federated module 没有 federated code 中所需的依赖，Webpack 将会从 federated 构建源中下载缺少的依赖项。

### Why

1. 方便代码共享
2. 共享依赖（代码最下化）
3. 运行时的代码共享可加快构建速度

### How

#### 基本概念

Host：消费其他 Remote 的 Webpack 构建；

Remote：被 Host 消费的 Webpack 构建；

![avatar](/images/host_remote.png)

一个构建产物可以同时是 Host 和 Remote，Remote 依赖的三方包会优先使用 host 里面加载了的，当 host 没有或者版本不一致的时候会加载自己的（当然也可以配置成版本不强制一致）

#### 配置

```
new ModuleFederationPlugin({
    name: 'name',                   // 模块名称
    filename: 'remoteEntry.js',     // 模块入口
    remotes: {                      // 引用模块
        app1: 'app1@http://localhost:3002/remoteEntry.js',  // 地址可以是绝对的或者相对的，@前面的符号会是remote提供模块在window下输出的变量
        app2: 'app2@http://localhost:3003/remoteEntry.js',
    },
    exposes: {                      // 导出模块
        './Layout': './src/Layout',
    },
    shared: {                       // 共享模块
        ...deps,
        react: { singleton: true, requiredVersion: deps.react },
        'react-dom': {
            singleton: true,        // 版本是否需要唯一（当配置为true的时候通知只能存在一个版本包，例如react、react-dom）
            eager: true,            // 是否同步加载（入口依赖的三方包需要同步加载，所有一般我们会在入口地方使用import方法）
            requiredVersion: deps['react-dom'],  // 依赖版本-如果可以任意版本可以设置成false
        },
    },
}),
```

## 示例

[示例地址](https://github.com/wangxiaofeid/webpack5-micro-frontends)

### 三个应用（basic、app1、app2）

basic：基座应用-部署时的主应用、包含部分公用页面、全局 store、通用样式等

app1：演示父子通讯、跳转、多版本包

app2：演示 remoteEntry.js 懒加载

### 优势

1. 微前端的优点
2. 构建产物最小
3. 简单（理解简单、开发方便）

### 问题

1. 样式冲突--命名空间、style-components
2. entry 多个--可优化（是否可以动态应用模块 entry[文档](https://webpack.docschina.org/concepts/module-federation/#dynamic-remote-containers)）

# 参考

[官方文档](https://webpack.docschina.org/concepts/module-federation)

[官方 Demo](https://github.com/module-federation/module-federation-examples)

[Webpack 5 Module Federation: JavaScript 架构的变革者](https://zhuanlan.zhihu.com/p/120462530)
