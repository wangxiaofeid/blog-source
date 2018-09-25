---
title: React优化笔记
date: 2018-09-07 16:57:08
tags:
---

平时看一些react文章，都有些性能相关的建议，以此记录。

#### react diff
文章地址[不可思议的 react diff](https://zhuanlan.zhihu.com/p/20346379)
* 保持稳定的 DOM 结构会有助于性能的提升
* 尽量减少类似将最后一个节点移动到列表首部的操作
* 同一级别下节点尽量都设置key（key必须是稳定的），可以减少节点create和remove
* 父级组件render必然触发子级的render，从而带来diff开销，可以合理的利用shoudComponentUpdate或者PureComponent
* component拆分总结
  * 交互频繁的组件和渲染数据量特别大组件的隔离


#### js层面
* 元素属性如果是写死的，可以提出来写在component外面，或者使用component的static属性（避免每次渲染都从新定义一个）
* 事件函数今天直接绑定component下的箭头函数，除非需要参数
* 组件内部方法如果可以的话尽量放到util内，或者使用static方法（避免每次渲染都实例化）

#### 构建层面
* 使用`export NODE_ENV=production`生成环境模式打包上线代码
* 使用prepack-webpack-plugin进行代码优化，预执行js代码
* 使用SplitChunksPlugin进一步对chunk之间的公共引用提取出来，可根据公共部分大小来判断是否提取
* mini-css-extract-plugin和SplitChunksPlugin，进行css分包