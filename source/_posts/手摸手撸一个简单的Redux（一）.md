---
title: 手摸手撸一个简单的Redux（一）
date: 2018-02-26 10:14:41
banner: http://img.yanyuanfe.cn/photo-1432821596592-e2c18b78144f.jpeg
tags:
 - Redux
 - React
---

> 理解Redux的原理有助于我们更好的使用它。本文实现Redux的基本API。

![image](http://img.yanyuanfe.cn/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67.png)

<!--more-->



Redux 试图让 state 的变化变得可预测。

redux已经被越来越多的人使用，理解其原理有助于更好的使用它。阅读redux源码是一个不错的办法，当我们了解了其原理之后，现在来实现一个简单的redux吧。

> 本文完整代码请查看Github：https://github.com/YanYuanFE/redux-app

``` bash
// clone repo
git clone https://github.com/YanYuanFE/redux-app.git


cd redux-app

// checkout branch
git checkout part-3

// install
npm install

// start
npm start

```

### 基本API
如果你使用过redux，应该对redux的API了然于胸吧。redux的基本API包括createStore、getState、subscribe、dispatch。现在回顾一下redux的使用方法：

首先，redux暴露出createStore方法，调用createStore方法传入reducer，使用const store = createStore(reducer)，（此处暂时不考虑中间件），使用store.getState()获取状态，使用store.dispatch()发起一个action，使用store.subscribe()订阅状态改变时执行的函数。下面是redux API的简单实现。

为了方便调试，使用前面讲的redux实现简单计数器的代码，在src目录下新建redux.js，

``` js
export function createStore(reducer) {

}
```

在redux.js中，先导出redux的核心方法 createStore，传入reducer作为参数。
在createStore方法中，需要定义用于保存当前状态的变量以及用于保存状态改变后执行函数的监听器。


``` js
let currentState;
let currentListeners = [];
```

此处定义currentState为当前状态，初始化为undefined;定义currentListenners为监听器，初始化为数组。

然后，定义getState方法，用于获取当前状态，直接返回currentState：

``` js
function getState() {
  return currentState;
}
```


然后，定义subscribe函数，用于订阅状态改变时执行的方法：

``` js
function subscribe(listener) {
  currentListeners.push(listener);
}
```

subscribe方法传入一个监听函数，将监听函数push进监听器数组中。
然后定义dispatch方法，用于发起action：

``` js
function dispatch(action) {
  currentState = reducer(currentState, action);
  currentListeners.forEach(v => v());
  return action;
}
```

dispatch方法传入action，然后调用reducer开始更新currentState，传入当前currentState和action，当状态改变时，通知监听器，监听器数组依次执行数组中的每一个订阅方法，此处使用了设计模式中的发布——订阅模式，然后返回action。

可以看到，上面已经实现了Redux的基本API，那么就结束了吗？当然没有，因为redux并没有初始化，reducer中的初始状态并没有生效，所以需要手动发起一个action，并且action.type必须是独一无二的。如下：

``` js
dispatch({type: '@@REDUX/INIT'});
```

此处，对redux进行初始化，定义type为'@@REDUX/INIT'，这样定义的原因是保证命中reducer中action.type为default使其返回初始化state。
最后，根据redux使用方法可以知道，store肯定是一个对象，对象包含getState、subscribe、dispatch等方法，所有，最后需要将上述方法返回。

``` js
return { getState, subscribe, dispatch };
```


至此，一个超级简单的redux就实现了，麻雀虽小，五脏俱全。这个redux虽然简单，省去了许多错误处理过程，但是对于理解redux足矣。下面是完整代码：

``` js
export function createStore(reducer) {
  let currentState;

  let currentListeners = [];

  function getState() {
    return currentState;
  }
  function subscribe(listener) {
    currentListeners.push(listener);
  }
  function dispatch(action) {
    currentState = reducer(currentState, action);
    currentListeners.forEach(v => v());
    return action;
  }

  dispatch({type: '@@REDUX/INIT'}); //初始化
  return { getState, subscribe, dispatch }
}
```

### 测试
下面结合之前的计数器例子来验证上述redux是否正确。
在前面计数器例子中，打开src下index.js，修改如下代码：

``` js
import { createStore } from './redux';
```


替换redux为刚刚编写的redux.js文件。查看浏览器运行结果。
![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_184.png)

如图，与redux的效果一致，达到预期效果。

### 总结
本文实现了一个简单的redux，完成其基本API的实现，这有助于你理解redux的原理，后面会逐步对其扩展，编写react-redux以及中间件的实现。