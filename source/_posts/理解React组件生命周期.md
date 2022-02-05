---
title: 理解React组件生命周期
date: 2017-01-24 22:10:27
categories: 前端
banner: http://img.yanyuanfe.cn/react_illo_800x600_1x.png
tags:
	- React
---


> 了解组件生命周期将使你能够在创建或销毁组件时执行某些操作。 此外，它让您有机会决定是否应该首先更新组件，并相应地对属性（props）或状态（state）更改做出反应。

![image](http://img.yanyuanfe.cn/react_illo_800x600_1x.png)

<!--more-->

<div class="tip">
    在公司实习已经一个月了，第一个项目是使用React+React-Redux架构的Webapp，在项目过程中对React的理解从入门到逐渐加深，所以写下这篇文章总结一下心得。
</div>

### 生命周期
React通过调用React.createClass（）方法来创建组件，React.createClass（）方法需要一个render方法，并触发一个生命周期，它可以通过一些所谓的生命周期方法来实现。

为了清楚地理解生命周期，我们需要区分组件被创建的初始创建阶段，状态和属性变化触发的更新阶段以及组件被卸载的阶段。

### 初始化（Initialization）
![image](http://img.yanyuanfe.cn/initial.png)

从上面的图我们可以看到，前两个方法被调用的是getDefaultProps和getInitialState。这两个方法只在最初渲染组件时调用一次。 getInitialState方法允许设置初始状态值，这是通过this.state在组件内部可访问的。


```js
getInitialState: function(){
    return { /* something here */};
}
```


类似地，getDefaultProps可以用于定义可以通过this.props访问的任何默认属性。

```js
getDefaultProps: function(){
    return { /* something here */};
}
```

另一个只有在初始化组件时才调用的方法是componentWillMount和componentDidMount。  
  
  **componentWillMount**在执行render方法之前被调用。重要的是要注意，在此阶段中设置状态不会触发重新渲染。  
  
**render**方法返回所需的组件标记，它可以是单个子组件或null或false（以防您不想要任何渲染）。  
这是生命周期中解释属性和状态值以创建正确输出的部分。在这个函数里面不应该修改属性和状态。这一点很重要，因为根据定义，render必须是纯函数，这意味着每次调用该方法时返回相同的结果。  
  
  一旦render方法被执行，**componentDidMount**函数被调用。在此方法中可以访问DOM，从而能够定义DOM操作或数据请求操作。任何DOM交互应该总是发生在这个阶段，而不是render方法内部。

### 状态变化（State Changes）
状态更改将触发许多钩子函数。
![image](http://img.yanyuanfe.cn/statechange.png)  

**shouldComponentUpdate**总是在render方法之前调用，并允许定义是否需要或跳过重新渲染。显然，这种方法从来没有在初始渲染时调用。必须返回布尔值。  

```js
shouldComponentUpdate: function(nextProps, nextState){
    // return a boolean value
    return true;
}
```
此方法可以访问当前以及即将到来的属性和状态确保可以检测到可能的改变以确定是否需要渲染。  
shouldComponentUpdate方法 允许我们手动地判断是否要进行组件更新，根据组件的应用场景设置函数的合理返回值能够帮我们避免不必要的更新。很多人不了解这个方法而不知道它的重要性。

**componentWillUpdate**在shouldComponentUpdate返回true时立即被调用。不允许通过this.setState进行任何状态更改，因为此方法应严格用于准备即将到来的更新而不触发更新本身。  

```js
componentWillUpdate: function(nextProps, nextState){
    // perform any preparations for an upcoming update
}
```
最后，在render方法之后调用**componentDidUpdate**。与componentDidMount类似，此方法可用于在数据更新后执行DOM操作。

```js
componentDidUpdate: function(prevProps, prevState){
    // 
}
```
### 属性变更（Props Changes）
对props对象的任何更改也将触发生命周期函数，并且几乎与状态更改相同，且有一个额外的方法被调用。
![image](http://img.yanyuanfe.cn/changeprops.png)  
**componentWillReceiveProps**仅在属性已更改并且不是初始渲染时调用。 componentWillReceiveProps允许根据现在和即将到来的属性来更新状态，而不触发另一个渲染。这里需要记住的一个有趣的事情是，没有等效的状态的方法，因为状态变化不应该触发任何属性的变化。

```js
componentWillReceiveProps: function(nextProps) {
  this.setState({
    // set something 
  });
}
```
剩余的生命周期函数与状态改变触发方法相同，在这里没有什么不同。

### 卸载（Unmounting）
![image](http://img.yanyuanfe.cn/unmount.png)  

组件卸载会触发的唯一一个方法是**componentWillUnmount**，当组件从DOM中移除之前调用componentWillUnmount。当需要执行清理操作时，该方法可能是有益的。删除componentDidMount中定义的任何计时器。

### 思考

> 在生命周期中的哪一步你应该发起 AJAX 请求？  

我们应当将AJAX 请求放到 componentDidMount 函数中执行，主要原因有下：  

- React 下一代调和算法 Fiber 会通过开始或停止渲染的方式优化应用性能，其会影响到 componentWillMount 的触发次数。对于 componentWillMount 这个生命周期函数的调用次数会变得不确定，React 可能会多次频繁调用 componentWillMount。如果我们将 AJAX 请求放到 componentWillMount 函数中，那么显而易见其会被触发多次，自然也就不是好的选择。

- 如果我们将 AJAX 请求放置在生命周期的其他函数中，我们并不能保证请求仅在组件挂载完毕后才会要求响应。如果我们的数据请求在组件挂载之前就完成，并且调用了setState函数将数据添加到组件状态中，对于未挂载的组件则会报错。而在 componentDidMount 函数中进行 AJAX 请求则能有效避免这个问题。

### 参考资料
- [React官方文档](https://facebook.github.io/react/docs/react-component.html)
