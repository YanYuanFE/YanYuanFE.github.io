---
title: React Router V4学习笔记
date: 2018-02-02 11:27:57
banner: http://img.yanyuanfe.cn/stock-photo-245282361.jpg
tags:
 - React Router
 - React
---

> React Router现在已经是React开发单页应用必备技能，自升级V4版本以来，许多核心API都进行了重写，践行路由即组件的概念，本文是我学习React Router V4以来的一些总结。

![image](http://img.yanyuanfe.cn/recatrouter.png)

<!--more-->

### 介绍
React-Router V4是react-router的一次重大更新，许多核心API都已改变，使用react-router4不需要对路由进行集中式配置，react-router 4的核心是一切皆组件。 

react-router 4一共被拆分为三个包：react-router、react-router-dom、react-router-native，react-router提供了React Router的核心路由功能，包括Router, Route, Switch等，但是我们不会直接使用它。react-router-dom和react-router-native提供在特定运行环境下的路由组件。如果你需要开发在浏览器中运行的Web应用，则应该安装react-router-dom。同样，如果你需要开发React Native应用程序，则应该安装react-router-native。这两者都将安装react-router作为依赖项。

### react-router-dom

react-router使用react-router-dom作为浏览器端的路由，提供了BrowserRouter,Route,Link等api,通过DOM的事件控制路由。在Web开发过程中，我们更多是使用react-router-dom。本文的讨论也仅限于react-router-dom。

**安装**

``` bash
npm install react-router-dom --save
```

### 使用

#### Router

react-router4的使用首先需要选择Router，Router用于包裹在应用最外层。Router有两种类型：HashRouter和BrowserRouter，其表现方式也有所不同，使用时需要区分。

#### HashRouter与BrowserRouter

从表面区分HashRouter和BrowserRouter的方法就是HashRouter的url中有个#，例如localhost:3000/#，HashRouter通过hash值来对路由进行控制。如果你使用HashRouter，你的路由就会默认有这个#。  

BrowserRouter的url中没有#，它的url如http://localhost:3001/user/login，BrowserRouter的原理是依赖HTML5的history API实现的路由机制。  

HashRouter和BrowserRouter都具有basename属性，如果你的URL不是位于网站根目录，而是放在子目录里，你需要设置这个属性。如下：


``` js
<BrowserRouter basename="/calendar"/>
<Link to="/today"/> // 渲染为 <a href="/calendar/today">
```

#### Route
Route是Router的子元素，控制路径对应显示的组件。常用属性有exact、path以及component。
考虑以下代码：

``` js
<Route path="/" exact component={HomePage} />
<Route path="/users" component={UsersPage} />
```
react-router4的路由是包含性的，多个Route可以同时进行匹配和渲染，exact控制到路径匹配成功时不会再继续向下匹配，在面代码中，我们试图根据路径渲染 HomePage 或者 UsersPage。如果从Route删除了 exact 属性，那么在浏览器中访问 /users 时，HomePage 和 UsersPage 组件将同时被渲染。

#### Switch

在Router组件中的任意位置创建多个<Route>，但通常我们会把它们放在同一个位置。使用<Switch>组件来包裹一组<Route>。<Switch>会遍历自身的子元素（即路由）并对第一个匹配当前路径的元素进行渲染。考虑如下代码：


``` js
<Switch>
  <Route exact path="/" component={DashBoard} />
  <Route path="/user" component={User} />
</Switch>
```

如果去掉Switch组件，当我们访问/user的时候，路由会同时匹配“/”和/user，将同时渲染DashBoard和User组件。

#### Link vs NavLink

react-router4提供了两种导航API，用于页面切换，当页面切换时，页面不会重新加载，但是组件会被重新渲染。它们作用相同，但NavLink在匹配URL成功时，可以提供一些额外的样式能力。

**Link**

Link提供to属性控制跳转地址，可传入字符串或者对象。如下：

``` js
<Link to="/courses"/>to: object
<Link to={{
  pathname: '/courses',
  search: '?sort=name',
  hash: '#the-hash',
  state: { fromDashboard: true }
}}/>
```
**NavLink**

NavLink可以给当前匹配成功的链接提供一个className类名，常在Tab导航中使用。


``` js
<nav>
  <NavLinkto="/app"exact activeClassName="active">Home</NavLink>
  <NavLinkto="/app/users"activeClassName="active">Users</NavLink>
  <NavLinkto="/app/products"activeClassName="active">Products</NavLink>
</nav>
```


#### 路径参数
在react router4中，match用于获取路径上的参数，match是使用<Route>渲染时传递到组件内的一个props，在react组件内部通过this.props.match获取match的属性，
考虑如下代码：

``` js
<Switch>
  <Route exact path="/" component={DashBoard} />
  <Route path="/user/:id" component={User} />
  <Route path="/user" component={User} />
</Switch>
```

当访问http://localhost:3001/user/123时，打印this.props.match的内容。
match获取的属性主要有：
![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_167.png)


然后，通过this.props.match.params.id就可以获取路由匹配的id了。

#### withRouter

试想，如果组件没有通过<Route>渲染，该如何获取history、match、location等props呢？react router4提供了一个高阶组件withRouter。使用方法如下：

``` js
import { withRouter } from 'react-router-dom';
```

使用withRouter有两种办法，第一种简洁的方法是使用装饰器，如下：

``` js
@withRouter
class AuthRoute extends React.Component {
}
```

另一种方法直接调用withRouter包裹原来的组件返回一个新的组件：

``` js
class Auth extends React.Component {

}
const AuthRoute = withRouter(Auth)
```

通过以上两种方法，就能在组件内部使用路由的props了。如下：

``` js
const { match，location，history} = this.props;
```


### 结语
关于react router4,本文只是讲解了一些常用的api和用法，需要学习的还有很多，这里只是开始。

### 参考

[react router 官方文档](http://reacttraining.cn/)