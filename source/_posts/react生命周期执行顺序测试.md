---
title: react生命周期执行顺序测试
date: 2018-09-04 19:23:23
tags: react lifecycle
---

项目地址：[https://github.com/wangxiaofeid/react-lifecycle](https://github.com/wangxiaofeid/react-lifecycle)

## 测试
### 首次渲染
```
father constructor
father getDerivedStateFromProps
father render
children constructor
children getDerivedStateFromProps
children render
children componentDidMount
father componentDidMount
```

### 父组件数据修改触发重渲染
```
father getDerivedStateFromProps
father shouldComponentUpdate
father render
children getDerivedStateFromProps
children shouldComponentUpdate
children render
children getSnapshotBeforeUpdate
father getSnapshotBeforeUpdate
children componentDidUpdate, snapshot: 1
father componentDidUpdate, snapshot: 1
```

### 父组件调用forceUpdate
```
father getDerivedStateFromProps
father render
children getDerivedStateFromProps
children shouldComponentUpdate
children render
children getSnapshotBeforeUpdate
father getSnapshotBeforeUpdate
children componentDidUpdate, snapshot: 1
father componentDidUpdate, snapshot: 1
```

### 销毁
```
father componentWillUnmount
children componentWillUnmount
```

## 总结
### 旧生命周期在各个阶段的调用情况
1. 挂载
    * constructor
    * componentWillMount
    * render
    * componentDidMount
2. 更新
    * componentWillReceiveProps
    * shouldComponentUpdate
    * componentWillUpdate
    * render
    * componentDidUpdate
3. 卸载
    * componentWillUnmount

### 新生命周期在各个阶段的调用情况
1. 挂载
    * constructor
    * getDerivedStateFromProps
    * render
    * componentDidMount
2. 更新
    * getDerivedStateFromProps
    * shouldComponentUpdate
    * render
    * getSnapshotBeforeUpdate
    * componentDidUpdate
3. 卸载
    * componentWillUnmount

### 父子组件之间的生命周期函数的调用顺序
* 挂载阶段，只有当执行到render的时候，父组件的constructor才开始执行，知道子组件挂载完成（componentDidMount），父组件才算挂载完成
* 更新阶段，类似挂载阶段，只有父组件执行到render，才开始子组件的getDerivedStateFromProps -> shouldComponentUpdate -> render，但再父组件的getSnapshotBeforeUpdate是紧随在子组件的getSnapshotBeforeUpdate后，然后子组件在componentDidUpdate

### 父组件调用forceUpdate
组件调用forceUpdate方法后，不会执行shouldComponentUpdate，会执行getDerivedStateFromProps，然后再render，后面的生命周期和更新一致