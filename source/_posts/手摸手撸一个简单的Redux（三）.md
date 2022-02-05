---
title: 手摸手撸一个简单的Redux（三）
date: 2018-03-8 10:36:15
banner: http://img.yanyuanfe.cn/photo-1432821596592-e2c18b78144f.jpeg
tags:
 - Redux
 - React
---

> It provides a third-party extension point between dispatching an
action, and the moment it reaches the reducer. ---中间件

![image](http://img.yanyuanfe.cn/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67.png)

<!--more-->

> 本文完整代码请查看Github：https://github.com/YanYuanFE/redux-app

``` bash
// clone repo
git clone https://github.com/YanYuanFE/redux-app.git


cd redux-app

// checkout branch
git checkout part-5

// install
npm install

// start
npm start

```


当我们的业务需求变得更为复杂的时候，单纯的在dispatch和reducer中处理业务逻辑已经不具有普遍性。我们需要的是可以组合的，自由插拔的插件机制，redux借鉴koa的中间件思想，实现了redux的middleware。在发出action和执行reducer之间，使用中间件函数对store.dispatch进行改造，所以，redux的middleware是为了增强dispatch而生的。

回顾中间件的使用方法，这里以，redux-thunk中间件为例，为之前的计数器添加一个延迟计数的功能，即点击按钮两秒后进行加1操作，redux和react-redux使用官方实现的，下面是改造后的代码：
首先是计数器组件，src下components目录下，Counter.js中，代码如下：

``` js
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class Counter extends Component {
  constructor(props) {
    super(props);

  }


  render() {
    const { value, onIncrement, onDecrement, incrementAsync } = this.props;
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
        </p>
    )
  }
}



Counter.propTypes = {
    value: PropTypes.number.isRequired,
    onIncrement: PropTypes.func.isRequired,
    onDecrement: PropTypes.func.isRequired,
    incrementAsync: PropTypes.func.isRequired
};

export default Counter;

```

上述代码中，添加了用于异步加1的按钮，点击触发props中的incrementAsync方法。
下面是src下App.js的代码：


``` js
import React, { Component } from 'react';
import { connect } from 'react-redux';
import Counter from './components/Counter';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    const { onIncrement, onDecrement, counter, incrementAsync } = this.props;

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

function onDecrement() {
  return { type: 'DECREMENT' }
}

function incrementAsync() {
  return dispatch => {
    setTimeout(() => {
      dispatch(onIncrement());
    }, 2000)
  }
}

const mapDispatchToProps = {
  onIncrement,
  onDecrement,
  incrementAsync
};

export default connect(mapStateToProps, mapDispatchToProps)(App);
```


相比上一篇文章，主要定义了incrementAsync方法，然后添加到mapDispatch中，在incrementSync中，返回一个函数，在函数内部执行异步操作发起action，普通的action都是一个对象的形式，但是异步的action返回的是一个function，处理这种情况就需要使用redux的中间件：redux-thunk。
下面是src下index.js的代码：

``` js
import React from 'react';
import ReactDOM from 'react-dom';
import thunk from 'redux-thunk';
import { Provider } from 'react-redux';
import './index.css';
import App from './App';
import {applyMiddleware, createStore} from 'redux';
import counter from './reducers';

const store = createStore(counter, applyMiddleware(thunk));

ReactDOM.render(
  <Provider store={store}>
    <App/>
  </Provider>,
  document.getElementById('root')
);
```


在index.js中，引入react-thunk模块，然后在createStore中传入第二个参数，使用redux的API applymiddleware对chunk中间件进行包裹。
上述就是改进后的计数器，运行项目在浏览器预览下，点击incrementAsync按钮，达到预期效果。如下图：

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_195.png)

### applyMiddleware实现

applyMiddleware是redux提供的用于使用中间件的API，回顾applyMiddleware的使用：

``` js
const store = createStore(counter, applyMiddleware(logger, thunk));
```
applyMiddleware接收多个中间件参数，返回值作为第二个参数传入createStore。

在src下redux.js文件中，首先让原来的createStore支持传入第二个参数，代码如下：

``` js
export function createStore(reducer, enhancer) {
  if (enhancer) {
    return enhancer(createStore)(reducer);
  }
}
```

在createStore中传入第二个参数enhancer，enhancer即调用applyMiddleware包装中间件的函数，然后判断enhancer是否存在，存在即调用enhancer传入createStore和reducer两个参数，由此，applyMiddleware应该是一个高阶函数，返回一个新的函数。
下面暴露applyMiddleware方法。redux的middleware是支持多个中间件的，此处先实现支持一个中间件的用法。

``` js
export function applyMiddleware(middleware) {
  return createStore => (...args) => {
    const store = createStore(...args);
    let dispatch = store.dispatch;

    const midApi = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    dispatch = middleware(midApi)(store.dispatch)
    return {
      ...store,
      dispatch
    }
  }
}
```

applyMiddleware的结构是一个多层柯里化的函数，第一层函数执行后返回一个函数，这个函数即createStore函数中的参数enhancer，然后这个函数传入createStore参数，再返回一个函数，这个函数传入reducer参数，使用...args进行解构，在函数内部，首先调用createStore获取到原始的store以及dispatch，然后封装一个对象midApi传入中间件内部，midApi包括两个方法getState和dispatch，getState对应store.getState方法，dispatch对应store.dispatch方法并透传参数。下面是一个日志中间件的代码：

``` js
const logger = store => next => action => {
 console.log('dispatching', action);
 let result = next(action);
 console.log('next state', store.getState());
 return result;
};
```


由此可见，中间件函数是一个层层包裹的匿名函数，第一层传入store，第二层传入next下一个中间件，此处指store.dispatch，第三层是在组件中进行调用时，传入action。logger中间件在applyMiddleware中被层层调用，动态的对store和next参数赋值。

接着看上面applyMiddleware的代码，定义了一个由getState和dispatch组成的闭包midApi，中间件函数middleware第一次调用传入midApi返回一个匿名函数，如下：
``` js
next => action => {
 console.log('dispatching', action);
 let result = next(action);
 console.log('next state', store.getState());
 return result;
};
```
紧接着对匿名函数再次调用，传入store.dispatch作为参数next，再次返回一个匿名函数，如下：
``` js
action => {
 console.log('dispatching', action);
 let result = next(action);
 console.log('next state', store.getState());
 return result;
};
```
通过对middleware的层层调用来生成一个新的dispatch方法，新的dispatch
对store原有的dispatch方法进行了增强，最后返回一个对象，使用解构赋值将增强的dispatch覆盖原有的store.dispatch，成为一个新的store。最终，在组件中发起dispatch的时候使用的就是新的dispatch方法。

到这里，已经让redux支持中间件的用法了，现在来使用一下，还是使用上面项目中的计数器案例，将redux和react-redux替换为自己编写的文件，redux-thunk不变，运行项目，依次点击三个按钮，达到预期效果。

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_196.png)

### 编写redux-thunk中间件
上面已经让我们的redux支持使用中间件，下面来尝试自己编写一个thunk中间件。在src下新建thunk.js：

``` js
const thunk = ({dispatch, getState}) => next => action => {

  return next(action)
}

export default thunk;
```

一个中间件的结构如上，一个三层箭头函数，第一个参数传入midApi，这里解构出dispatch和getState方法方便使用，第二个参数传入下一个中间件，即store.dispatch，第三个参数传入需要提交的action。在中间件函数中，如果什么都不做，直接返回next（action），即直接dispatch action。当然，这样做并没有什么用，下面为它加上thunk的代码：

``` js
const thunk = ({dispatch, getState}) => next => action => {
  // 如果是函数，执行一下
  if (typeof action === 'function') {
    return action(dispatch, getState);
  }
  return next(action)
}

export default thunk;
```

这就是redux-thunk的代码，很简单吧，首先判断传入的action是否为fuction，如果是function就执行该action，并传入dispatch和getState， 如下，在incrementAsync的返回函数的参数中，可以接受dispatch和getState两个参数：

``` js
function incrementAsync() {
  return (dispatch, getState) => {
    setTimeout(() => {
      dispatch(onIncrement());
    }, 2000)
  }
}
```

下面来验证一下实现的thunk中间件，在src下替换redux-thunk为./thunk。在浏览器依次点击三个按钮，结果如下，达到预期效果。

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_197.png)