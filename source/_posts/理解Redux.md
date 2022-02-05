---
title: 理解Redux
date: 2018-02-06 14:30:55
banner: http://img.yanyuanfe.cn/stock-photo-245270081.jpg
tags:
  - React
  - Redux
---

> Redux的核心思想是Web应用是一个状态机，视图与状态是一一对应的。

![image](http://img.yanyuanfe.cn/redux.png)

<!--more-->

> 最近一年的工作中都在使用React相关的技术栈，当初理解Redux也费了很大功夫，一直都想写一点关于Redux的教程，现在，终于开始了。

### 为什么使用Redux

Redux的出现是为了解决Javascript中复杂的状态管理，当你的应用变得庞大，需要处理复杂的交互场景时，你可以尝试使用Redux。如果你使用React，如果你需要共享状态到很多组件中，如果你需要处理异步，如果一个组件的状态改变需要更新其他组件，你可能需要Redux。

### 介绍
Redux 是 JavaScript 状态容器，提供可预测化的状态管理。

Redux专注于状态管理，和react解耦。

Redux 可以用这三个基本原则来描述：

- 单一状态树

整个应用的 state 被储存在一棵对象树中，并且这个对象树只存在于唯一一个store中。

- State 是只读的

唯一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象。

- 使用纯函数来执行修改

为了描述 action 如何改变 state树，你需要编写 reducers。

### Action

Action 本质上是 JavaScript 普通对象。Redux约定，action 内必须使用一个字符串类型的 type 字段来表示将要执行的动作。多数情况下，type 会被定义成字符串常量。当应用规模越来越大时，建议使用单独的模块或文件来存放action。

Redux 中只需把 action 创建函数的结果传给 dispatch() 方法即可发起一次 dispatch 过程。
store 里能直接通过 store.dispatch() 调用 dispatch() 方法。

### Reducer

Reducers 根据传入的action和state计算新的state。

Reducer 就是一个纯函数，接收旧的 state 和 action，返回新的 state。
只要传入参数相同，计算得到的下一个 state 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。

1. reducer中不要修改state，而是返回新对象。 可以使用 Object.assign() 新建一个副本。不能这样使用：

``` js
Object.assign(state, { visibilityFilter: action.filter })
```
上述代码会改变第一个参数的值。必须把第一个参数设置为空对象。也可以开启ES7使用对象展开运算符, 从而使用 { ...state, ...newState } 达到相同的目的。
3. 在 default 情况下返回旧的 state。遇到未知的 action 时，一定要返回旧的 state。

## Store

store 是维持应用state的容器，并在当你发起 action 的时候调用 reducer。
Store 就是把它们联系到一起的对象。Store 有以下职责：
- 维持应用的 state；
- 提供 getState() 方法获取 state；
- 提供 dispatch(action) 方法更新 state；
- 通过 subscribe(listener) 注册监听器;
- 通过 subscribe(listener)返回的函数注销监听器。

Redux 应用只有一个单一的 store。当需要拆分数据处理逻辑时，你应该使用 多个reducer 组合而不是创建多个 store。

### 数据流

Redux 应用中数据的生命周期遵循下面 4 个步骤：
1. 调用 store.dispatch(action)。

你可以在任何地方调用 store.dispatch(action)，包括组件中、XHR 回调中、甚至定时器中。

2. Redux store 调用传入的 reducer 函数。

Store 会把两个参数传入 reducer： 当前的 state 树和 action。

3. 根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树。

4. Redux store 保存了根 reducer 返回的完整 state 树。

### 总结

本文只是一些关于redux的概念和基本介绍，下次会带来它的基本用法。