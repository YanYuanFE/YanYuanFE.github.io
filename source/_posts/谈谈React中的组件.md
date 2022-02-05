---
title: 谈谈React中的组件
date: 2018-03-19 21:00:56
banner: http://img.yanyuanfe.cn/photo-1454165804606-c3d57bc86b40%20%281%29.jpeg
tags:
- React
---

> React的核心思想是一切皆组件

![image](http://img.yanyuanfe.cn/react.png)

<!--more-->

在React中，一切皆组件。组件可以将UI切分为一系列独立的、可复用的部件，这样使我们可以专注于构建每一个单独的部件。

理解React中各种各样的组件对于更好的学习React变得尤为重要，如UI组件、容器组件、无状态组件、函数式组件，对react了解不够深入的人通常对于这些概念理解不够清晰，很难写出高质量的react组件，本文是我对于React的组件的一些理解。

### Component

Compoent在这里指React.Component，大多数人一开始学习React了解最多的便是Component。使用Component可以非常方便的定义一个组件。
使用ES6的class来定义一个组件如下：


``` js
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```


### PureComponent

PureComponent在这里指React.PureComponent，PureComponent与Component几乎完全相同，但是PureComponent通过props和state的浅对比来实现shouldComponentUpdate（）。

``` js
class Welcome extends React.PureComponent {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

在React的生命周期中，当组件的props和state发生改变时，会执行组件的shouldComponentUpdate函数，当该函数返回为true时组件重新渲染，返回为false则不重新渲染。使用Component定义的组件中，shouldComponentUpdate默认返回为true，通常当组件遭遇性能问题时，可以在shouldComponent中对新旧属性和状态进行相等判断，来减少不必要的重新渲染。

PureComponent是react v15.3.0中新加入的特性，默认实现了shouldComponentUpdate中对于新旧属性和状态的相等比较，减少不必要的重新渲染来提升性能。

注意：PureComponent的shouldComponentUpdate只会对对象进行浅比较，当对象层级较深或者结构复杂时，可能会出现对象深层数据已改变而视图没有更新的问题，使用时需要注意。

### 函数式组件

上述使用Component和PureComponent定义的组件称为声明式组件。在react中，还可以使用函数定义一个组件，称为函数式组件。如下：


``` js
const Text = ({ children = 'Hello World!' }) =>
  <p>{children}</p>
```

函数式组件又称无状态（stateless）组件，在组件中不能对状态（state）进行操作，使用函数式组件使react的代码更具有可读性，可以减少诸如class、constructor等冗余代码，有利于组件复用。
函数式组件具有以下特点：

1. 函数式组件不会被实例化，提升了整体渲染性能。

函数式组件没有class声明，仅使用render方法实现，不存在组件实例化的过程，因此不需要分配多余的内存，从而提升了渲染性能。

2. 函数式组件不能访问this对象。

函数式组件内部不能访问this.state，无法操作状态。
3. 函数式组件没有生命周期。
4. 函数式组件只接受props和context参数，没有副作用。

### UI组件和容器组件

在结合redux管理数据流的应用中，redux将组件分为UI组件（presentational component）和容器组件（container component）。UI组件又称为展示型组件。在早期的Redux的版本中，作者将上述两种组件定义为智能（Smart）组件和木桶（Dumb）组件，两者的区别是看是否有数据操作。Dumb组件对应UI组件，Smart组件对应容器组件。UI组件只负责UI的呈现，不负责业务逻辑;内部没有状态，数据由this.props参数提供;容器组件负责处理业务逻辑，不负责UI呈现;有内部状态;使用Redux的API连接UI组件。

通常在项目的文件结构中，components文件夹包含UI组件，containers文件夹包含容器组件。

### 纯组件
提到纯组件，先来回顾一下函数式编程中的“纯函数”，纯函数由三大原则构成：
- 给定相同的输入，总是返回相同的输出;
- 函数执行过程中不会产生副作用（side effect）;
- 没有额外的状态依赖;

在react中，使用纯组件可以提高Virtual DOM的执行效率，那么，怎样的组件才是纯组件呢？

首先，PureComponent并不是纯组件，因为在组件中存在生命周期，并且拥有内部状态state，可以产生副作用。

通常，函数式组件、UI组件、无状态组件这类仅接受props参数进行数据渲染的组件可以称为纯组件，它们接受相同的输入参数props，返回相同的输出，且不具有副作用。

### 受控组件
在HTML中，像input,textarea, 和 select这类表单元素会维持自身状态，并根据用户输入进行更新。但在React中，可变的状态通常保存在组件的状态属性中，并且只能用 setState(). 方法进行更新。

在react中，使用state来控制表单元素的数据的显示，同时控制表单元素输入引起值的变化并更新state来进行表单元素数据的更新，其值由React控制的输入表单元素称为“受控组件”。
考虑如下代码：


``` js
class Form extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

上述代码是一个受控组件的示例，由于 value 属性是在我们的表单元素上设置的，因此显示的值将始终为 React数据源上this.state.value 的值。由于每次按键都会触发 handleChange 来更新当前React的state，所展示的值也会随着不同用户的输入而更新。

### 非受控组件

在大多数情况下，我们推荐使用 受控组件 来实现表单。 在受控组件中表单数据由 React 组件处理。如果让表单数据由 DOM 处理时，替代方案为使用非受控组件。

要编写一个非受控组件，而非为每个状态更新编写事件处理程序，你可以 使用 ref 从 DOM 获取表单值。
考虑如下代码：

``` js
class Form extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={(input) => this.input = input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

 由于非受控组件将真实数据保存在 DOM 中，因此在使用非受控组件时，更容易同时集成 React 和非 React 代码。如果你想快速而随性，这样做可以减小代码量。否则，你应该使用受控组件。

通常，不建议使用非受控组件。

### 高阶组件

高阶组件（HOC）是react中对组件逻辑进行重用的高级技术。但高阶组本身并不是React API。它只是一种模式，这种模式是由react自身的组合性质必然产生的。

具体而言，高阶组件类似于高阶函数，且该函数接受一个React组件作为参数，并返回一个新的组件。


``` js
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```


你或许没听过高阶组件之前已经使用过了，在react的一些第三方库，如react-redux的connect，react-router-dom的withRouter等等。

高阶组件可以让代码更具有复用性、逻辑性与抽象特性。它可以对render方法做劫持，也可以控制props与state。

考虑以下代码：

``` js
export default function Form(Comp) {
  return class WrapperComp extends Component {
    constructor(props) {
      super(props);
      this.state = {

      }
      this.handleChange = this.handleChange.bind(this);
    }

    handleChange(key, val) {
      this.setState({
        [key]: val
      })
    }

    render() {
      return <Comp {...this.props} handleChange={this.handleChange} state={this.state}/>
    }
  }
}
```

上述代码是一个高阶组件，为了解决使用表单组件时频繁的编写时间处理函数来管理表单数据的问题。使用方法如下：

``` js
class Login extends Component {
  constructor(props) {
    super(props);
    this.state = {

    }
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(key, val) {
    this.setState({
      [key]: val
    })
  }

  render() {
    return (
      <div className="login">
        <input
          type="text"
          value={this.props.state.user}
          onChange={(e) => this.props.handleChange('user', e.target.value)}
        />
        <input
          type="password"
          value={this.props.state.psw}
          onChange={(e) => this.props.handleChange('psw', e.target.value)}
        />
      </div>
    )
  }
}

export default Form(Login);
```


这样，对于需要在很多地方使用表单的组件，可以提供组件复用度，减少样板代码。



> 参考：https://react.bootcss.com/