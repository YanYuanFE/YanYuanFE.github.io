---
title: 编写高性能的React代码
date: 2018-9-25 21:56:56
banner: http://img.yanyuanfe.cn/react.jpg
tags:
- React
---
> 本文总结了一些编写React代码的建议，有助于理解React的渲染机制，主要是通过减少不必要的重新渲染来提升React性能。

![image](http://img.yanyuanfe.cn/react.jpg)

<!--more-->

回顾使用React开发的经历，已有接近两年了，一直想要写一篇文章来介绍如何编写高性能的React代码，又或者是性能调优之类的，于是，我总结了一些编写高性能React代码的实践，或许能让你对React的理解更加深刻。

### 影响React性能的主要因素
React专注于构建用户界面，内部通过VirtualDOM和Diff算法实现视图的更新，本身拥有很高的性能。当组件的state和props改变的时候，React就会重新render，进而更新视图。在render过程中，react会将JSX语法转换为虚拟DOM，同时进行Diff算法比较新旧DOM的变化，从而对视图进行局部更新。试想，当render执行后，JSX转换虚拟DOM，然后Diff，最后发现新旧DOM是一样的，最后并没有更新视图。那么，就导致了不必要的render，造成了性能的浪费。所以，减少不必要的render，成为优化React性能的主要因素。

### state只与视图有关
编写React组件的时候，你肯定会用到变量，变量可以保存在this上，可以保存到state中，还可以使用全局变量。那么什么时候该使用state呢？答案是只有与视图相关的变量才放在state中，与视图无关的可以放在this上。如果与视图更新无关的变量放在state中，而且这个变量需要频繁的更新，而每次更新都需要重新render，势必造成性能浪费，使用this来存放无疑是更好的选择。

### shouldComponentUpdate
考虑如下代码：

``` js
export default class Home extends Component {
  state = {
    count: 0
  }

  add = () => {
    this.setState({count: 0});
  }
  
  render() {
    console.log('render');
    return (
      <div className="App">
        <div onClick={this.add}>+++</div>
        <p className="App-intro">
          {this.state.count}
        </p>
      </div>
    );
  }
}
```
当点击+++触发add事时，setState更新state，但是新旧state完全一样，React并没有在Component组件中进行优化，导致不必要的render执行。此时，需要我们手动进行优化，当state或者props更新后react组件的生命周期shouldComponentUpdate触发，在shouldComponentUpdate中可以对新旧state和props进行比较，判断组件是否需要重新render。

``` js
shouldComponentUpdate(nextProps, nextState) {
    if(nextState.count === this.state.count) {
      return false;
    }
}
```
默认情况下shouldComponentUpdate返回true，即始终要重新渲染，此处我们使用shouldComponentUpdate进行比较后，返回false，react将不会重新render。

当state为字符串等基本类型的时候，shouldComponentUpdate还能应付自如，那如果state是对象或者数组等引用类型呢？对于引用类型只能对其进行递归比较才能判断其是否相等，又或者是采用JSON.stringify(nextState.obj) === JSON.stringify(this.state.obj)进行比较，但是当对象嵌套层级较深或者函数、正则等类型时，shouldComponentUpdate便失去了它的用武之地。

### PureComponent
React.PureComponent与React.Component 几乎完全相同，但React.PureComponent 通过prop和state的浅对比来实现 shouldComponentUpate()。
简而言之，PureComponent自己内部实现了一个shouldComponentUpdate，而不需要我们再去编写比较代码。
React.PureComponent的shouldComponentUpdate() 只会对对象进行浅对比。如果对象包含复杂的数据结构，它可能会因深层的数据不一致而产生错误的否定判断(表现为对象深层的数据已改变视图却没有更新）。
测试表明，使用PureComponent的性能会略胜shouldComponentUpdate一筹。


### StateLessComponent
无状态组件（StateLessComponent）又称函数式组件。

``` js
const Text = ({ children = 'Hello World!' }) =>
  <p>{children}</p>
```
函数式组件没有class声明，没有生命周期，仅使用render方法实现，不存在组件实例化的过程，因此不需要分配多余的内存，从而提升了渲染性能。


### 箭头函数与bind
当我们在组件上绑定事件处理函数的时候，为了能正常访问state，通常会使用bind或者箭头函数。

```js
<Button onClick={this.update.bind(this)} />
```
又或者

``` js
<Button onClick={() => { console.log("Click"); }} />
```

在以上情况下，bind每次都返回全新的函数，箭头函数在每次 render 时都会重新分配（和使用 bind 的方式相同），对于组件来说每次绑定的都是新的函数，Button组件在进行 props 比较的时候会认为前后props不一样，造成重新渲染。
比较推荐的写法有两种：

``` js
update = () => {
    //
}
<Button onClick={this.update} />
```
或者
``` js
constructor(props) {
    super(props);
    this.update = this.update.bind(this);
}

update() {
    //
}
<Button onClick={this.update} />
```

### 内联style
当我们为组件设置样式的时候，可能会写出如下代码：


``` js
<Button style={{width: 100, marginTop: 50 }} />
```
这种写法，每次render时style返回的都是全新的对象，我们知道，引用类型比较的是引用地址，每次的结果都是不相等，也会导致重新渲染。
推荐的写法是使用className。
```js
<Button className="btn" />
```

### 属性传递与[]

当我们为组件设置缺省值的时候:

``` js
<RadioGroup options={this.props.options || []} />
```

如果每次 this.props.options 值都是 null 的话，意味着每次传递给<RadioGroup />都是字面量数组[]，但字面量数组和new Array()效果是一样的，始终生成新的实例，所以表面上看虽然每次传递给组件的都是相同的空数组，其实对组件来说每次都是新的属性，都会引起渲染。所以正确的方式应该将一些常用值以变量的形式保存下来：

``` js
const DEFAULT_OPTIONS = [];

<RadioGroup options={this.props.options || DEFAULT_OPTIONS} />
```
<div class="tip">
注意：此处的DEFAULT_OPTIONS不能写在render中，当写在render中的时候，还是会每次都生成一个新的数组，依然会重新渲染，正确的做法是放在组件外面或者使用this.DEFAULT_OPTIONS = []。
</div>

### map 与 key
在react中，你应该经常用到map来渲染一个列表，当你忘记为每个列表元素设置key属性的时候，react会在控制台发出警告a key should be provided for list items。
实际上，Keys可以在DOM中的某些元素被增加或删除的时候帮助React识别哪些元素发生了变化。因此你应当给数组中的每一个元素赋予一个确定的标识。

一个元素的key最好是这个元素在列表中拥有的一个独一无二的字符串。通常，我们使用来自数据的id作为元素的key:

``` js
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

当元素没有确定的id时，你可以使用他的序列号索引index作为key：

``` js
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```
当todoItems中数据发生改变的时候，使用索引作为key的列表都将重新渲染。所以不建议使用索引来进行排序，因为这会导致渲染变得很慢。

### 减少在render内执行变量

我们已经知道，当组件的state和props发生改变的时候，render都会执行，那么，当render中有变量的执行或者函数执行的时候，可能会影响渲染性能，所以，应该减少在render内执行变量或者函数。

### 精简props和state
从上面我们已经知道，当props和state为引用类型的时候，react内部只对props和state进行浅比较，那么，props和state可以尽量设置为基础数据类型，对象的嵌套层级不要过深，减少props的参数数量，使组件的props和state对象数据尽可能达到扁平化，都可以在一定程度上优化react的渲染性能。

### 总结

本文讨论了一系列编写React代码的原则，旨在从减少重新render的方面来提升react代码的性能。性能优化一直是一个持久的话题，还可以从很多方面去考虑，比如，使用不可变的数据结构immutable.js，使用reselect来缓存redux中的复杂计算，对组件尽可能进行拆分为子组件，都是值得探讨的话题。那么，下次再见。
