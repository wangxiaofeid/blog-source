---
title: 基于vue后端管理系统的最佳实践
date: 2018-07-09 14:40:57
tags: vue 后台
---

### 为什么要写这么个东西
* 后台管理做的多了，提炼一些通用性东西用在以后工作中
* 希望懂点前端的后端开发能在这个项目的基础上，写更少的内容来实现日常的增删改查
* 加深对mixins，vuex等理解

### 目录结构
```
  -- build
  -- config
  -- dist
  -- src
     |-- assets                // 静态资源
     |-- commonPages         // 通用页面（首页，登录等）
     |-- components          // 通用component
     |-- filters               // filters
     |-- layout               // 整体布局
     |-- mixins               // mixins
     |-- pages                // 页面代码
     |-- router               // 主路由
     |-- store                 // store文件夹
     |-- styles                // 样式文件夹
     |-- utils                  // 工具方法
     |-- ajax.js              // 统一的ajax请求入口，方便做登录校验等通用功能
     |-- App.vue            // App.vue
     |-- main.js             // 打包入口
  -- static
  -- template
  -- index.html
  -- package.json
```

### 如何使用
``` bash
/* clone 到本地 */
git clone https://github.com/wangxiaofeid/vue-admin-best

/* 安装依赖 */
yarn

/* 本地启动 */
npm start

/* 打包 */
npm run build
```

### 如何新建一个页面
``` bash
npm run cleate <pageName>
``` 
根据设置的名称一次性创建页面component, router, store, edit弹窗；
store为modules里面的子store，名称约定为`${pageName}Store`，开启命名空间；

### 注意事项
* 页面记录列表默认唯一key为`id`，如果表数据不是id，可以在component的data里面设置rowKey，在store初始化时传rowKey参数
* 页面添加完不能直接在菜单里显示，需要在主store里添加
* 页面数据的增删改查接口风格尽量统一，以便做统一逻辑
* 本例使用了淘宝的[rap2](http://rap2.taobao.org/repository/editor?id=18404&itf=133875)，可以理解为部署在服务端的mock.js的实现

### 不足待改进的地方
