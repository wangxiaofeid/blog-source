---
title: react服务端渲染
date: 2018-07-21 17:19:43
tags:
---

很久之前写的一个项目[https://github.com/wangxiaofeid/reactServerRender](https://github.com/wangxiaofeid/reactServerRender)

## 技术栈
webpack express react react-router mobx less等

## 遇到的一些问题
* 服务端渲染出来的html和客户端第一次渲染的html不一致是会有警告
* mobx的store转化成json时不能深度解析，使用mobx.toJS(store, true)也不行，自己做了层转化
* 服务端使用match后，客户端也要使用match，不然渲染出来的html不一致，会有警告
* 客户端使用match后点击其他页面后，html修改了url不改变的问题，使用mobx-react-router里的RouterStore，再对react-router的Link做一层封装，具体看代码

## 简单说下怎么做的
1. 每个页面有一个store，store都有一个init方法，一个isLoad标记是否初始化
2. 每个页面入口的componentDidMount方法里调用页面pageStore.init()
3. init方法内返回promise，请求全部成功返回后设置isLoad=true
4. 从后端到前端的流程是：
   * 后端收到请求
   * 匹配路由
   * await pageStore.init() 等待页面数据返回
   * 渲染模板，并把store数据塞到html
   * 前端js执行，初始化store，把window下数据初始化到各个pageStore
   * 渲染页面component
   * 执行pageStore.init()，判断是否已经初始化了
   * end