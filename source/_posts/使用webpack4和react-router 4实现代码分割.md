---
title: 使用webpack4和react-router 4实现代码分割
date: 2018-03-30 14:05:16
banner: http://img.yanyuanfe.cn/photo-1416339442236-8ceb164046f8.jpeg
tags:
- React
- react-router
- Webpack
---

> Webpack的Code Splitting特性能够把代码分离到不同的bundle中，然后可以按需加载或并行加载这些文件。

![image](http://img.yanyuanfe.cn/banner_1025.jpg)

<!--more-->

### 什么是代码分割（code splitting）
我们知道，在使用webpack打包react应用时，webpack将整个应用打包成一个js文件，当用户访问首屏时，会一次性加载整个js文件，这就造成了首屏渲染速度变慢的问题。于是，webpack开发了代码分割的特性，
此特性能够把代码分离到不同的 bundle 中，然后可以按需加载或并行加载这些文件。代码分离可以用于获取更小的 bundle，以及控制资源加载优先级，如果使用合理，会极大影响加载时间。
有三种常用的代码分离方法：
* 入口起点：使用 entry 配置手动地分离代码。
* 防止重复：使用 CommonsChunkPlugin 去重和分离 chunk。
* 动态导入：通过模块的内联函数调用来分离代码。
本文只讨论动态导入（dynamic imports）的方法。

### 动态导入
当涉及到动态代码拆分时，webpack 提供了两个类似的技术。对于动态导入，第一种，也是优先选择的方式是，使用符合 ECMAScript 提案 的 import() 语法。第二种，则是使用 webpack 特定的 require.ensure。让我们先尝试使用第一种……

> 注意：import() 调用会在内部用到 promises。如果在旧有版本浏览器中使用 import()，记得使用 一个 polyfill 库（例如 es6-promise 或 promise-polyfill），来 shim Promise。

下面结合react-router 4来实现react的代码分割。

### react code splitting and lazy load
在使用react-router4进行代码分割的路上，社区已经有成熟的第三方库进行了实现，如react-loadable。在此处将介绍如何不借助第三方库实现代码分割。

此处我们假设你已经对react、react-router4、webpack有基本的了解，可以搭建简单的开发环境。下面先看一下我搭建的项目入口文件。

src下index.js：

``` js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';


ReactDOM.render(<App />, document.getElementById('app'));
src下App.js。
import React from 'react';
import ReactDOM from 'react-dom';
import { Route, Switch, BrowserRouter } from 'react-router-dom';
import Routes from './routes';




export default () => (
  <BrowserRouter>
    <Switch>
      {
        Routes.map(({name, path, exact = true, component }) => (
          <Route path={path} exact={exact} component={component} key={name} />
        ))
      }
    </Switch>
  </BrowserRouter>
)
```

在App.js中，使用了react-router-dom来做路由，routes.js文件作为路由配置文件，下面使用路由配置遍历生成路由组件。
下面来看一下路由配置routes.js。

``` js
import AsyncComponent from './components/acync-component';

export default [
  {
    name: '首页',
    icon: 'home',
    path: '/',
    component: AsyncComponent(() => import('./containers/home'))
  },
  {
    name: '详情',
    icon: 'detail',
    path: '/detail',
    component: AsyncComponent(() => import('./containers/detail'))
  }
]
```

routes.js中配置了路由组件需要的参数，需要注意的是在路由参数中使用了异步组件AsyncComponent，注意这里并没有进行组件的引入，而是传给了AsyncComponent一个函数，它将在 *AsyncComponent（() => import('./containers/home')）* 组件被创建时进行动态引入。同时，这种传入一个函数作为参数，而非直接传入一个字符串的写法能够让 webpack意识到此处需要进行代码分割。

使用import（）需要使用像 Babel预处理器和Syntax Dynamic Import Babel Plugin。由于 import() 会返回一个 promise，因此它可以和 async 函数一起使用，使用acync函数需要使用babel-plugin-transform-runtime。

安装babel插件：


``` bash
npm i -D babel-plugin-syntax-dynamic-import babel-plugin-transform-runtime
```


本项目使用的其他babel插件还有
babel-core、babel-loader、babel-preset-env、babel-preset-react等，主要用于jsx编译
配置.babelrc
根目录下新建.babelrc，配置如下：

```
{
  "presets":["react","env"],
  "plugins": ["syntax-dynamic-import", "transform-runtime"]
}
```

### 异步组件AsyncComponent
本项目的AsyncComponent放在src/components/async-component/index.js中，代码如下：

``` js
import React, { Component } from 'react';

export default (loadComponent, placeholder = '拼命加载中。。。。') => {
  return class AsyncComponent extends Component {
    

    constructor() {
      super();

      this.state = {
        Child: null
      }

      this.unmount = false;
    }

    componentWillUnmount() {
      this.unmount = true;
    }

    async componentDidMount() {
      const { default: Child } = await loadComponent();

      if (this.unmount) return;

      this.setState({
        Child
      })
    }

    render() {
      const { Child } = this.state;

      return (
        Child ? <Child {...this.props} /> : placeholder
      )
    }
  }
}
```

首先，这是一个高阶组件，返回一个新的组件，传入两个参数，一个是需要动态加载组件的方法，第二个是动态加载时的占位符，也可以传入一个Loading组件。

在返回的AsyncComponent内部，constructor中，初始化一个state为Child，值为null，并定义this.unmount =false，用于表示组件是否被卸载。

使用acync定义异步方法，componentDidMount中，使用await异步执行传入的第一个参数，用于动态加载当前路由的组件。

> 注意：
注意当调用 ES6 模块的 import() 方法（引入模块）时，必须指向模块的 .default 值，因为它才是 promise 被处理后返回的实际的 module 对象。

故此处使用ES6的对象解构获取到模块的default并赋值到Child上。
然后判断组件被卸载的状态，被卸载即返回。
下面将Child设置到state上。
在render方法中，从state中获取Child，然后使用三元运算符判断Child是否为true，为true则渲染Child组件，并注入this.props，否则渲染占位符。
组件componentWillUnmount时，设置this.unmout为true。

### 测试验证
在containers中新建两个文件夹home和detail，在两个文件夹下编写index.js作为两个路由组件。代码如下：
home/index.js：

``` js
import React from 'react';
import ReactDOM from 'react-dom';
import { Link } from 'react-router-dom';
import '../../style.css';
import icon from '../../assets/404.png';


export default class Home extends React.Component {
  render() {
    return (
      <div>
        <h1>hello webpack</h1>
        <img src={icon} alt=""/>
        <Link to="/detail">详情</Link>
      </div>
    )
  }
}
```


detail/index.js：


``` js
import React from 'react';
import ReactDOM from 'react-dom';
import '../../style.css';
import icon from '../../assets/404.png';

export default class Detail extends React.Component {
  render() {
    return (
      <div>
        <h1>hello webpack detail</h1>
        <img src={icon} alt=""/>
      </div>
    )
  }
}
```


在根目录下package.json配置启动脚本：

``` js
"scripts": {
    "start": "webpack-dev-server --mode development",
    "build": "webpack --mode production"
  },
```


然后运行npm start启动项目：
打开浏览器访问localhost：8080

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_217.png)

可以看到页面先加载了main.js和0.js，点击详情按钮跳转到http://localhost:8080/detail。

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_218.png)

马上加载了1.js，这样就实现了代码分割，每个路由都是动态加载的。提升了首屏渲染速度。