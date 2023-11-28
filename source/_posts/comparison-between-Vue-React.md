---
title: Vue 和 React 的比较
toc: true
categories:
  - Front-end Study
tags:
  - Vue
  - React
date: 2023-11-25 21:39:12
---

> 自从入了 Vue 的坑，都快忘记了 React

最近突然想起来 React，这么久没有接触，想试着玩一玩。

回想起大学时，学长们的分享，总是说 Vue 更易上手，所以就入了 Vue 的坑，但是仔细想想，自己却从来没有亲自去认真了解过 React，真是罪过...

趁此机会了解一下 React，试着把 Vue 和 React 进行一下比较。在网上找到一篇大佬的分享，觉得非常棒，文章内容很详细，链接我放到后文中。

我看完之后想简单总结性的记一下

<!-- more -->

## 异同点

Vue 和 React 都是用于构建用户界面的 JavaScript 库，它们都提供了组件化、声明式编码等特性。它们之间有一些相似点，但也有一些区别。

### 相同点

- 都使用 Virtural DOM
- 都使用组件化思想，流程基本一致
- 都是响应式，推崇单向数据流
- 都有成熟的社区，都支持服务端渲染

> Vue 和 React 实现原理和流程基本一致，都是使用 Virtual DOM + Diff 算法。

#### 通用流程

Vue 和 React 通用流程：

1. vue template/react jsx
2. render 函数
3. 生成 VNode
4. 当有变化时，新老 VNode diff -> diff 算法对比，并真正去更新真实 DOM。

#### Virtual DOM

`为什么Vue和React都选择Virtual DOM`（React 首创 VDOM，Vue2.0 开始引入 VDOM）？

1. 减少直接操作 DOM。框架给我们提供了屏蔽底层 dom 书写的方式，减少频繁的整更新 dom，同时也使得数据驱动视图
2. 为函数式 UI 编程提供可能（React 核心思想）
3. 可以跨平台，渲染到 DOM（web）之外的平台。比如 ReactNative，Weex

### 差异点

#### 核心思想不同

Vue推崇灵活易用（`渐进式`开发体验），`数据可变`，`双向数据绑定（依赖收集）`。

React的`函数式编程`这个基本盘不会变。

#### 组件实现不同

Vue源码实现是把options挂载到Vue核心类上，然后再new Vue({options})拿到实例（vue组件的script导出的是一个挂满options的纯对象而已）。

React内部实现比较简单，直接定义render函数以生成VNode，而`React内部使用了四大组件类包装VNode`，不同类型的VNode使用相应的组件类处理，职责划分清晰明了（后面的Diff算法也非常清晰）。

#### 响应式原理不同

`Vue依赖收集，自动优化`，数据可变。Vue递归监听data的所有属性,直接修改。当数据改变时，自动找到引用组件重新渲染。

`React基于状态机，手动优化`，数据不可变，需要setState驱动新的State替换老的State。当数据改变时，以组件为根目录，默认全部重新渲染

#### diff 算法不同

Vue Diff使用`双向链表`，边对比，边更新DOM。
React主要使用`diff队列`保存需要更新哪些DOM，得到patch树，再统一操作批量更新DOM

#### 事件机制不同

`Vue原生事件使用标准Web事件`。Vue组件自定义事件机制，是父子组件通信基础。Vue合理利用了snabbdom库的模块插件

`React原生事件被包装`，所有事件都冒泡到顶层document监听，然后在这里合成事件下发。基于这套，可以跨端使用事件机制，而不是和Web DOM强绑定。React组件上无事件，父子组件通信使用props

## 参考资料

[个人理解Vue和React区别](https://lq782655835.github.io/blogs/vue/diff-vue-vs-react.html)