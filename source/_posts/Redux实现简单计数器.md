---
title: Redux实现简单计数器
date: 2018-02-7 20:37:16
banner: http://img.yanyuanfe.cn/photo-1453928582365-b6ad33cbcf64.jpeg
tags:
 - Redux
---

> redux专注于状态管理，和react解耦，redux也可以结合Angular一起使用。本文结合实例讲解Redux的使用，可以更好的理解Redux。

![image](http://img.yanyuanfe.cn/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67.png)

<!--more-->

当你已经了解一些redux的基础概念后，现在开始尝试使用redux吧，redux专注于状态管理，和react解耦，redux也可以结合Angular一起使用。本文尝试在react应用中使用redux来开发应用，而并没有使用react-redux，这有助于理解redux的数据流。

> 本文完整代码请查看Github：https://github.com/YanYuanFE/redux-app

``` bash
// clone repo
git clone https://github.com/YanYuanFE/redux-app.git


cd redux-app

// checkout branch
git checkout part-1

// install
npm install

// start
npm start

```

### 开始
首先，初始化一个react应用，使用creat-react-app来进行创建，使用之前需要先安装，打开命令行，输入：

``` bash
npm i create-react-app -g
```
进行全局安装。然后进入准备开发的目录，打开命令行，进入当前文件夹，输入：

``` bash
create-react-app redux-app
```
，等待创建完成。然后

``` bash
cd redux-app
```

，运行npm start，react应用会自动运行并打开浏览器。
下一步，需要安装redux，命令行输入

``` bash
npm i redux --save
```

，下面，开始编写redux部分。
### redux
由于应用比较简单，可以直接编写reducer部分，在src目录下新建reducers文件夹，新建index.js，在reducer中，主要处理action来返回新的state，在本应用中，计数器触发的action主要有加和减两种，分别将其定义为
'INCREMENT'和'DECREMENT'，当触发计数器加1的时候，将state+1,触发计数器减1的时候，将state-1,否则，将返回原来的satte。并且需要将整个reducer export default以供其他部分使用。代码如下：

``` js
export default (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}
```

reducer编写完毕后，下一步需要生成store注入整个应用。此处，需要修改src目录下的index.js，通过import导入redux，需要使用redux提供的APIcreateStore将reducer生成整个应用的状态store，并且将store通过props传入整个应用的根组件App。同时，需要将原本的渲染逻辑封装成一个render方法并手动调用。同时，当store改变后，应用并不会自动重新渲染，需要使用store的方法subscribe来手动订阅，store.subscribe(render);意思是让store订阅render方法，当store改变时，就调用render重新渲染整个应用。


``` js
ReactDOM.render(<App store={store}/>, document.getElementById('root'))
```



``` js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { createStore } from 'redux';
import registerServiceWorker from './registerServiceWorker';
import counter from './reducers';

const store = createStore(counter);

const render = () => ReactDOM.render(<App store={store}/>, document.getElementById('root'));

render();

store.subscribe(render);
registerServiceWorker();
```

### 组件编写
store通过props传递到APP组件后，在APP组件内需要通过store来获取数据以及触发action。在APP组件中，为了使组件逻辑分离，计数器相关的实现将会单独抽离成组件，APP组件负责将数据和事件通过props传递到计数器组件中。
在src目录下新建components目录，在components目录下新建Counter.js，下面是计数器的代码：

``` js
import React, { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.incrementAsync = this.incrementAsync.bind(this);
    this.incrementIfOdd = this.incrementIfOdd.bind(this);
  }

  render() {
    const { value, onIncrement, onDecrement } = this.props;
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
      </p>
    )
  }
}

export default Counter;
```


在Counter组件中，通过props接收父组件传递的value和加、减数值的方法，value绑定到视图显示计数器的数值，加、减按钮分别用来控制计数器的加减操作，分别将父组件传递的方法进行绑定到按钮上。
然后在src目录下的App.js中引入计数器的组件，并在render中使用。

``` js
import React, { Component } from 'react';
import Counter from './components/Counter';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    const { store } = this.props;

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
          value={store.getState()}
          onIncrement={() => store.dispatch({ type: 'INCREMENT' })}
          onDecrement={() => store.dispatch({ type: 'DECREMENT' })}
        />
      </div>
    );
  }
}

export default App;
```

在APP组件中，使用ES6的对象解构语法在props中获取store，在render中渲染计数器组件Counter，同时，向Couter组件传入约定的props，计数器数值value通过store.getState()来获取，增加数值方法onIncrement传入一个方法，调用store.dispatch来发起一个Action，action中传入代表增加数值的type ‘Increment’，同理，减少数值的方法onDecrement中传入的actionType为‘DECREMENT’。

到这里，整个计数器应用就完全开发完成了。

``` js
npm start
```

启动应用，打开浏览器，最终效果如下：

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_208.png)

点击+、-按钮可以控制数字的加减，而且数字是实时更新的。

### 写在最后
通过redux实现的计数器应用已经开发完成了，你可以尝试梳理一下redux的流程，首先，redux的核心包括store、action、reducer，store暴露到全局应用中，通过store可以获取state，触发action，订阅state改变后的事件。要想改变state的数据，必须触发Action，action可以通过一些交互和事件来触发，reducer用来处理action，并返回一个新的state，当state改变后，store的subscribe方法就会触发，可以用来更新视图。如果你使用react来开发应用，当你的应用变得复杂，仅仅使用redux会变得很繁琐，你可能需要将store通过props来一层层传递到你需要的组件中，为了解决这样的问题，react-redux出现了，可以使用简单的方式获取到全局的状态，那么，拭目以待吧。