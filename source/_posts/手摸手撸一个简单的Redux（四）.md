---
title: 手摸手撸一个简单的Redux（四）
date: 2018-03-12 20:36:22
banner: http://img.yanyuanfe.cn/photo-1432821596592-e2c18b78144f.jpeg
tags:
 - Redux
 - React
---

> 理解Redux的原理有助于我们更好的使用它。本文实现redux的多个中间件合并功能。

![image](http://img.yanyuanfe.cn/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67.png)

<!--more-->


在上一篇文章中实现了redux的中间件机制，支持了传入一个中间件的用法，在实际的redux中，applyMiddleware是支持传入多个中间件的，本文使用redux实现多个中间件合并。

> 本文完整代码请查看Github：https://github.com/YanYuanFE/redux-app

``` bash
// clone repo
git clone https://github.com/YanYuanFE/redux-app.git


cd redux-app

// checkout branch
git checkout part-6

// install
npm install

// start
npm start

```

### 中间件合并

使用多个中间件的示例代码如下：


``` js
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);
```


还是在原来的项目中进行开发，在src下redux.js中，对applyMiddleware函数进行修改。

``` js
export function applyMiddleware(...middlewares) {
}
```
单个中间件middleware的结构如下：

``` js
store => next => action => {
 
 let result = next(action);
 
 return result;
};
```


当传入多个middlewares参数时，将参数展开，middlewares成为一个数组方便后面操作。



``` js
export function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args);
    let dispatch = store.dispatch;

    const midApi = {
      getState: store.getState(),
      dispatch: (...args) => dispatch(...args)
    }
    const middlewareChain = middlewares.map(middleware => middleware(midApi));
    dispatch = compose(...middlewareChain)(store.dispatch);
    return {
      ...store,
      dispatch
    }
  }
}
```

当有多个中间件时，对中间件数组middlewares执行map方法，对每个中间件都执行一次并传入midApi，返回成为一个新的数组middlewareChain。
middlewareChain中保存着middleware执行一次后返回的函数[mid1, mid2, mid3],每个mid的结构如下：

``` js
next => action => {
 
 let result = next(action);
 
 return result;
};
```

然后，需要一个compose方法来对每个mid方法进行依次执行，并返回一个函数，最后传入store.dispatch参数。
compose方法作用如下：


``` js
compose(fn1, fn2, fn3)
fn1(fn2(fn3)))
```


compose传入一系列函数作为参数，然后将一系列函数参数嵌套依次进行调用。
下面是compose的实现：


``` js
function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg;
  }
  if (funcs.length === 1) {
    return funcs[0]
  }
  return funcs.reduce((ret, item) => (...args) => ret(item(...args)))
}
```


在compose方法中，传入一系列函数参数，展开，funcs为一个数组。
当funcs.length为0时，即一个参数都没有的时候，返回一个默认函数;当传入一个参数时，直接返回第一个参数;当传入多个函数参数时，使用数组的reduce方法依次对funcs数组从左到右执行一个函数，该函数中，ret为上一次执行该函数的返回值，如果没有指定初始值，第一次执行时为数组第一个参数，item为当前正在处理的数组元素。
执行compose(fn1, fn2, fn3)，在reduce方法中的ret和item每次执行的结果如下：
第一次执行，ret为fn1,item为fn2,返回fn1（fn2（））;
第二次执行，ret为fn1（fn2（）），item为fn3,返回结果为fn1(fn2(fn3（）)))。

到此，redux现在已经支持多个中间件的用法了。

### 编写中间件进行测试
为了测试多个中间件，这里我们再编写一个简单的中间件用于支持数组action，在src目录下新建redux.array.js。
代码如下:

``` js
const arrayThunk = ({dispatch, getState}) => next => action => {
  if (Array.isArray(action)) {
    action.forEach(v => dispatch(v))
  }
  return next(action)
}

export default arrayThunk;
```


上述代码中，定义了arrayThunk中间件，使用Array.isArray方法判断action是否为数组，如果是数组就遍历action依次dispatch。
下面在原来的计数器应用中，增加一个按钮，使用arrayThunk中间件点击每次加2。
修改components/Counter.js如下：

``` js
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.incrementAsync = this.incrementAsync.bind(this);
    this.incrementIfOdd = this.incrementIfOdd.bind(this);
  }

  incrementIfOdd() {
    if (this.props.value % 2 !== 0) {
      this.props.onIncrement();
    }
  }

  incrementAsync() {
    setTimeout(this.props.onIncrement, 1000);
  }

  render() {
    const { value, onIncrement, onDecrement, incrementAsync, addTwice } = this.props;
    console.log(this.props);
    return (
      <p>
        Clicked: {value} times
        {' '}
        <button onClick={onIncrement}>
          +
        </button>
        {' '}
        <button onClick={onDecrement}>
          -
        </button>
        {' '}
        <button onClick={incrementAsync}>
          Increment async
        </button>
        {' '}
        <button onClick={addTwice}>
          +2
        </button>
      </p>
    )
  }
}

Counter.propTypes = {
  value: PropTypes.number.isRequired,
  onIncrement: PropTypes.func.isRequired,
  onDecrement: PropTypes.func.isRequired,
  incrementAsync: PropTypes.func.isRequired,
  addTwice: PropTypes.func.isRequired,
};

export default Counter;
```

在Counter.js中，增加一个按钮，点击触发props中的addTwice方法。

在src下，App.js中，修改代码如下：


``` js
import React, { Component } from 'react';
import { connect } from './react-redux';
import Counter from './components/Counter';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    const { onIncrement, onDecrement, counter, incrementAsync, addTwice } = this.props;

    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">Welcome to React</h1>
        </header>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
        <Counter
          value={counter}
          onIncrement={onIncrement}
          onDecrement={onDecrement}
          incrementAsync={incrementAsync}
          addTwice={addTwice}
        />
      </div>
    );
  }
}

const mapStateToProps = (state) => ({
  counter: state
});

function onIncrement() {
  return { type: 'INCREMENT' }
}

function addTwice() {
  return [{ type: 'INCREMENT' }, { type: 'INCREMENT' }]
}

function onDecrement() {
  return { type: 'DECREMENT' }
}

function incrementAsync() {
  return (dispatch, getState) => {
    setTimeout(() => {
      dispatch(onIncrement());
    }, 2000)
  }
}

const mapDispatchToProps = {
  onIncrement,
  onDecrement,
  incrementAsync,
  addTwice,
};

export default connect(mapStateToProps, mapDispatchToProps)(App);
```


增加了addTwice方法，发起一个数组action，在数组中，定义了两个增加计数的action，并加入mapDispatch中，在App组件中，从props中获取到addwice方法并传入Counter组件。

在index.js中，需要引入arrayThunk中间件。


``` js
import React from 'react';
import ReactDOM from 'react-dom';
import thunk from './thunk';
import arrThunk from './redux-array';
import { Provider } from './react-redux';
import './index.css';
import App from './App';
import {applyMiddleware, createStore} from './redux';

import counter from './reducers';


const store = createStore(counter, applyMiddleware(thunk, arrThunk));


ReactDOM.render(
  <Provider store={store}>
    <App/>
  </Provider>,
  document.getElementById('root')
);
```

在index.js中，引入arrayThunk并传入applyMiddleware中。

npm start启动项目，打开浏览器进行操作，结果如下，达到预期效果。

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_198.png)

