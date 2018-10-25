---
title: 【转】深入理解-React-高阶组件
date: 2018-10-25 11:34:55
tags:
---

在目前的前端社区，『推崇组合，不推荐继承（prefer composition than inheritance）』已经成为了比较好的实践，mixin 也因为自身的一些问题而渐渐不被推荐。高阶组件（Higher order components）作为 mixin 之外的一种组件抽象与处理形式，有哪些不同和好处呢？继续阅读来了解一下吧！

本文翻译自 franleplant 的博客 React Higher Order Components in depth (阅读原文及文中部分链接请自备梯子)

## 摘要
这篇文章的目标群体是有一定基础的并想要了解高阶组件的用户。如果你是 React 新手，那么可能你应该先阅读 React 的文档。

高阶组件是一个非常棒的形式，它已经在多个 React 库中被证明了它的价值。这篇文章中，我们将回顾什么是高阶组件，如何使用高阶组件，它的限制以及如何编写高阶组件。

附录中我们回顾了与高阶组件相关的话题（非核心），但是是我认为应当涉及到的知识。

这篇文章意在写的详尽，如果你发现了任何遗漏，请你报告它，我会做出相应的改变。

这篇文章假设读者拥有 ES6 的相关知识。

让我们开始吧！

## 什么是高阶组件？
一个高阶组件只是一个包装了另外一个 React 组件的 React 组件。

这种形式通常实现为一个函数，本质上是一个类工厂（class factory），它下方的函数标签伪代码启发自 Haskell

hocFactory:: W: React.Component => E: React.Component

这里 W（WrappedComponent） 指被包装的 React.Component，E（Enhanced Component） 指返回的新的高阶 React 组件。

定义中的『包装』一词故意被定义的比较模糊，因为它可以指两件事情：

1. 属性代理（Props Proxy）：高阶组件操控传递给 WrappedComponent 的 props，
2. 反向继承（Inheritance Inversion）：高阶组件继承（extends）WrappedComponent。
我们将讨论这两种形式的更多细节。

## 我可以使用高阶组件做什么呢？
概括的讲，高阶组件允许你做：

* 代码复用，逻辑抽象，抽离底层准备（bootstrap）代码
* 渲染劫持
* State 抽象和更改
* Props 更改
在探讨这些东西的细节之前，我们先学习如何实现一个高阶组件，因为实现方式『允许/限制』你可以通过高阶组件做哪些事情。

## 高阶组件工厂的实现
在这节中我们将学习两种主流的在 React 中实现高阶组件的方法：属性代理（Props Proxy）和 反向继承（Inheritance Inversion）。两种方法囊括了几种包装 WrappedComponent 的方法。

### Props Proxy （PP）
属性代理的实现方法如下：
```
function ppHOC(WrappedComponent) {
  return class PP extends React.Component {
    render() {
      return <WrappedComponent {...this.props}/>
    }
  }
}
```
可以看到，这里高阶组件的 render 方法返回了一个 type 为 WrappedComponent 的 React Element（也就是被包装的那个组件），我们把高阶组件收到的 props 传递给它，因此得名 Props Proxy。

注意：
```
<WrappedComponent {...this.props}/>
// is equivalent to
React.createElement(WrappedComponent, this.props, null)
```
它们都创建了一个 React Element，描述了 React 在『reconciliation』（可以理解为解析）阶段的渲染内容，如果你想了解更多关于 React Element 的内容，请看 Dan Abramov 的这篇博客 和官方文档上关于 reconciliation process 的部分。

#### Props Proxy 可以做什么？
* 更改 props
* 通过 refs 获取组件实例
* 抽象 state
* 把 WrappedComponent 与其它 elements 包装在一起
##### 更改 props
你可以『读取，添加，修改，删除』将要传递给 WrappedComponent 的 props。

在修改或删除重要 props 的时候要小心，你可能应该给高阶组件的 props 指定命名空间（namespace），以防破坏从外传递给 WrappedComponent 的 props。

例子：添加新 props。这个应用目前登陆的一个用户可以在 WrappedComponent 通过 this.props.user 获取
```
function ppHOC(WrappedComponent) {
  return class PP extends React.Component {
    render() {
      const newProps = {
        user: currentLoggedInUser
      }
      return <WrappedComponent {...this.props} {...newProps}/>
    }
  }
}
```
#####  通过 refs 获取组件实例
你可以通过 ref 获取关键词 this（WrappedComponent 的实例），但是想要它生效，必须先经历一次正常的渲染过程来让 ref 得到计算，这意味着你需要在高阶组件的 render 方法中返回 WrappedComponent，让 React 进行 reconciliation 过程，这之后你就通过 ref 获取到这个 WrappedComponent 的实例了。

例子：下方例子中，我们实现了通过 ref 获取 WrappedComponent 实例并调用实例方法。
```
function refsHOC(WrappedComponent) {
  return class RefsHOC extends React.Component {
    proc(wrappedComponentInstance) {
      wrappedComponentInstance.method()
    }
    render() {
      const props = Object.assign({}, this.props, {ref: this.proc.bind(this)})
      return <WrappedComponent {...props}/>
    }
  }
}
```
当 WrappedComponent 被渲染后，ref 上的回调函数 proc 将会执行，此时就有了这个 WrappedComponent 的实例的引用。这个可以用来『读取，添加』实例的 props 或用来执行实例方法。

##### 抽象 state
你可以通过向 WrappedComponent 传递 props 和 callbacks（回调函数）来抽象 state，这和 React 中另外一个组件构成思想 Presentational and Container Components 很相似。

例子：在下面这个抽象 state 的例子中，我们幼稚地（原话是naively :D）抽象出了 name input 的 value 和 onChange。我说这是幼稚的是因为这样写并不常见，但是你会理解到点。
```
function ppHOC(WrappedComponent) {
  return class PP extends React.Component {
    constructor(props) {
      super(props)
      this.state = {
        name: ''
      }
      this.onNameChange = this.onNameChange.bind(this)
    }
    onNameChange(event) {
      this.setState({
        name: event.target.value
      })
    }
    render() {
      const newProps = {
        name: {
          value: this.state.name,
          onChange: this.onNameChange
        }
      }
      return <WrappedComponent {...this.props} {...newProps}/>
    }
  }
}
```
然后这样使用它：
```
@ppHOC
class Example extends React.Component {
  render() {
    return <input name="name" {...this.props.name}/>
  }
}
```
这里的 input 自动成为一个受控的 input。

点击此链接查看一个更常见的双向绑定的高阶组件例子

##### 把 WrappedComponent 与其它 elements 包装在一起
出于操作样式、布局或其它目的，你可以将 WrappedComponent 与其它组件包装在一起。一些基本的用法也可以使用正常的父组件来实现（附录 B），但是就像之前所描述的，使用高阶组件你可以获得更多的灵活性。

例子：包装来操作样式
```
function ppHOC(WrappedComponent) {
  return class PP extends React.Component {
    render() {
      return (
        <div style={{display: 'block'}}>
          <WrappedComponent {...this.props}/>
        </div>
      )
    }
  }
}
```

### 反向继承（II）可以像这样简单地实现：
```
function iiHOC(WrappedComponent) {
  return class Enhancer extends WrappedComponent {
    render() {
      return super.render()
    }
  }
}
```
如你所见，返回的高阶组件类（Enhancer）继承了 WrappedComponent。这被叫做反向继承是因为 WrappedComponent 被动地被 Enhancer 继承，而不是 WrappedComponent 去继承 Enhancer。通过这种方式他们之间的关系倒转了。

反向继承允许高阶组件通过 this 关键词获取 WrappedComponent，意味着它可以获取到 state，props，组件生命周期（component lifecycle）钩子，以及渲染方法（render）。

我不会详细介绍你可以使用组件生命周期方法做什么，因为这是 React 的内容，而不是高阶组件的。但是请注意，你可以通过高阶组件来给 WrappedComponent 创建新的生命周期挂钩方法，别忘了调用 super.[lifecycleHook] 防止破坏 WrappedComponent。

#### Reconciliation 过程
介绍之前先来总结一些理论。

React Element 在 React 执行它的 reconciliation 的过程时描述什么将被渲染。

React Element 可以是两个种类其中的一种：String 或 Function。String 类型的 React Element 代表原声 DOM 节点，Function 类型的 React Element 代表通过 React.Component 创建的组件。想要了解更多关于 Elements 和 Components 的知识请阅读此推文。

Function 类型的 React Element 将在 reconciliation 阶段被解析成 DOM 类型的 React Element （最终结果一定都是 DOM 元素）。

这点非常重要，这意味着『反向继承的高阶组件不保证一定解析整个子元素树』。这对渲染劫持非常重要。

#### 可以用反向继承高阶组件做什么？
* 渲染劫持（Render Highjacking）
* 操作 state

##### 渲染劫持
它被叫做渲染劫持是因为高阶组件控制了 WrappedComponent 生成的渲染结果，并且可以做各种操作。

通过渲染劫持你可以：

* 『读取、添加、修改、删除』任何一个将被渲染的 React Element 的 props
* 在渲染方法中读取或更改 React Elements tree，也就是 WrappedComponent 的 children
* 根据条件不同，选择性的渲染子树
* 给子树里的元素变更样式

*渲染 指的是 WrappedComponent.render 方法

你无法更改或创建 props 给 WrappedComponent 实例，因为 React 不允许变更一个组件收到的 props，但是你可以在 render 方法里更改子元素/子组件们的 props。

就像之前所说的，反向继承的高阶组件不能保证一定渲染整个子元素树，这同时也给渲染劫持增添了一些限制。通过反向继承，你只能劫持 WrappedComponent 渲染的元素，这意味着如果 WrappedComponent 的子元素里有 Function 类型的 React Element，你不能劫持这个元素里面的子元素树的渲染。

例子1：条件性渲染。如果 this.props.loggedIn 是 true，这个高阶组件会原封不动地渲染 WrappedComponent，如果不是 true 则不渲染（假设此组件会收到 loggedIn 的 prop）
```
function iiHOC(WrappedComponent) {
  return class Enhancer extends WrappedComponent {
    render() {
      if (this.props.loggedIn) {
        return super.render()
      } else {
        return null
      }
    }
  }
}
```
例子2：通过 render 来变成 React Elements tree 的结果
```
function iiHOC(WrappedComponent) {
  return class Enhancer extends WrappedComponent {
    render() {
      const elementsTree = super.render()
      let newProps = {};
      if (elementsTree && elementsTree.type === 'input') {
        newProps = {value: 'may the force be with you'}
      }
      const props = Object.assign({}, elementsTree.props, newProps)
      const newElementsTree = React.cloneElement(elementsTree, props, elementsTree.props.children)
      return newElementsTree
    }
  }
}
```
在这个例子中，如果 WrappedComponent 的顶层元素是一个 input，则改变它的值为 “may the force be with you”。

这里你可以做任何操作，比如你可以遍历整个 element tree 然后变更某些元素的 props。这恰好就是 Radium 的工作方式。

注意：你不能通过 Props Proxy 来做渲染劫持

即使你可以通过 WrappedComponent.prototype.render 获取它的 render 方法，你需要自己手动模拟整个实例以及生命周期方法，而不是依靠 React，这是不值当的，应该使用反向继承来做到渲染劫持。要记住 React 在内部处理组件的实例，而你只通过 this 或 refs 来处理实例。

##### 操作 state
高阶组件可以 『读取、修改、删除』WrappedComponent 实例的 state，如果需要也可以添加新的 state。需要记住的是，你在弄乱 WrappedComponent 的 state，可能会导致破坏一些东西。通常不建议使用高阶组件来读取或添加 state，添加 state 需要使用命名空间来防止与 WrappedComponent 的 state 冲突。

例子：通过显示 WrappedComponent 的 props 和 state 来 debug
```
export function IIHOCDEBUGGER(WrappedComponent) {
  return class II extends WrappedComponent {
    render() {
      return (
        <div>
          <h2>HOC Debugger Component</h2>
          <p>Props</p> <pre>{JSON.stringify(this.props, null, 2)}</pre>
          <p>State</p><pre>{JSON.stringify(this.state, null, 2)}</pre>
          {super.render()}
        </div>
      )
    }
  }
}
```
## 命名
当通过高阶组件来包装一个组件时，你会丢失原先 WrappedComponent 的名字，可能会给开发和 debug 造成影响。

常见的解决方法是在原先的 WrappedComponent 的名字前面添加一个前缀。下面这个方法是从 React-Redux 中拿来的。
```
HOC.displayName = `HOC(${getDisplayName(WrappedComponent)})`
//or
class HOC extends ... {
  static displayName = `HOC(${getDisplayName(WrappedComponent)})`
  ...
}
```
方法 getDisplayName 被如下定义：
```
function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || 
         WrappedComponent.name || 
         ‘Component’
}
```
实际上你不用自己写这个方法，因为 recompose 库已经提供了。

## 案例学习
React-Redux
React-Redux 是 Redux 官方的对于 React 的绑定。 其中一个方法 connect 处理了所有关于监听 store 的 bootstrap 代码 以及清理工作，这是通过 Props Proxy 来实现的。

如果你曾经使用过 Flux 你会知道 React 组件需要和一个或多个 store 连接，并且添加/删除对 store 的监听，从中选择需要的那部分 state。而 React-Redux 帮你把它们实现了，自己就不用再去写这些了。

Radium
Radium 是一个增强了行内（inline）css 能力的库，它允许了在 inline css 使用 CSS 伪选择器。点击此链接了解关于使用 inline css 的好处，这是 Vjeux 做的一个演讲分享，又叫做 CSS in JS。

那么，Radium 是怎么允许 inline css 来实现 CSS 伪选择器的呢（比如 hover）？它实现了一个反向继承来使用渲染劫持，添加适当的事件监听来模拟 CSS 伪选择器。这要求 Radium 读取整个 WrappedComponent 将要渲染的元素树，每当找个某个元素带有 style prop，它就添加对应的时间监听 props。简单地说，Radium 修改了原先元素树的 props（实际上会更复杂，但这么说你可以理解到要点所在）。

Radium 只暴露了一个非常简单的 API 给开发者。这非常惊艳，因为开发者几乎不会注意到它的存在和它是怎么发挥作用的，而实现了想要的功能。这揭露了高阶组件的能力。

## 附录 A：高阶组件和参数
以下内容不是必须阅读的，你可以略过。

有时，在高阶组件中使用参数是很有用的。这个在以上所有例子中都不是很明显，但是对于中等的 JavaScript 开发者是比较自然的事情。让我们迅速的介绍一下。

例子：一个简单的 Props Proxy 高阶组件搭配参数。重点是这个 HOCFactoryFactory 方法。
```
function HOCFactoryFactory(...params) {
  // do something with params
  return function HOCFactory(WrappedComponent) {
    return class HOC extends React.Component {
      render() {
        return <WrappedComponent {...this.props}/>
      }
    }
  }
}
```
你可以这样使用它：
```
HOCFactoryFactory(params)(WrappedComponent)
//or
@HOCFatoryFactory(params)
class WrappedComponent extends React.Component{}
```
## 附录 B：和父组件的不同之处
以下内容不是必须阅读的，你可以略过。

父组件就是单纯的 React 组件包含了一些子组件（children）。React 提供了获取和操作一个组件的 children 的 APIs。

例子：父组件获取它的 children
```
class Parent extends React.Component {
  render() {
    return (
      <div>
        {this.props.children}
      </div>
    )
  }
}

render((
  <Parent>
    {children}
  </Parent>
), mountNode)
```
现在来总结一下父组件能做和不能做的事情（与高阶组件对比）：

* 渲染劫持
* 操作内部 props
* 抽象 state。但是有缺点，不能再父组件外获取到它的 state，除非明确地实现了钩子。
* 与新的 React Element 包装。这似乎是唯一一点，使用父组件要比高阶组件强，但高阶组件也同样可以实现。
* Children 的操控。如果 children 不是单一 root，则需要多添加一层来包括所有 children，可能会使你的 markup 变得有点笨重。使用高阶组件可以保证单一 root。
* 父组件可以在元素树立随意使用，它们不像高阶组件一样限制于一个组件。
通常来讲，能使用父组件达到的效果，尽量不要用高阶组件，因为高阶组件是一种更 hack 的方法，但同时也有更高的灵活性。

## 总结
希望你在读完这篇文章后，能对 React 高阶组件多一丝了解。它们在多个库中被证明非常有效。

React 带来了很多创新，人们维护着像 Radium，React-Redux，React-Router 之类的项目，都是很好的证明。

如果你想联系我，请在 Twitter 关注我并 @franleplant。

这个仓库里有我为了写这篇文章而写的一些代码，有兴趣可以看看。

作者：Wenliang
链接：https://www.jianshu.com/p/0aae7d4d9bc1
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。