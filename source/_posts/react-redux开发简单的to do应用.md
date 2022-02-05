---
title: react-redux开发简单的To do应用
date: 2018-02-23 16:44:49
banner: http://img.yanyuanfe.cn/photo-1454165804606-c3d57bc86b40%20%281%29.jpeg
tags:
 - Redux
 - React
---

> redux专注于状态管理，和react解耦，为了方便使用，redux的作者封装了一个react专用的库react-redux。

![image](http://img.yanyuanfe.cn/bg2016092101.jpg)

<!--more-->

前面通过一个简单的计数器应用来学习redux的基本原理和用法，在本篇文章，将介绍redux如何与react结合使用。
为了方便使用，redux的作者封装了一个react专用的库react-redux，这也是接下来的重点。

> 本文完整代码请查看Github：https://github.com/YanYuanFE/redux-app

``` bash
// clone repo
git clone https://github.com/YanYuanFE/redux-app.git


cd redux-app

// checkout branch
git checkout part-2

// install
npm install

// start
npm start

```

### 安装


``` bash
npm install react-redux --save
```


### 基本API

#### <Provider/>组件
react-redux提供Provider组件，包裹在根组件最外层，用于传递store到组件内部，让应用内部的任何子孙组件非常方便地获取到全局状态，核心原理是react提供的context API。
示例代码如下：


``` js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import './index.css';
import App from './App';
import { createStore } from 'redux';
import reducer from './reducers';

const store = createStore(reducer);

ReactDOM.render(
  <Provider store={store}>
    <App/>
  </Provider>,
  document.getElementById('root')
);
```

#### connect()
react-redux提供connect方法用于将react组件与redux的store进行连接,通过connect方法，可以在任何react组件中，将store中的全局state和Action Creator传递到组件的props中，以供组件使用。

connect方法接受两个参数，**mapStateToProps**和**mapDispatchToProps**，用于定义组件的输入输出。mapStateToProps负责输入逻辑，将state映射到组件的props，后者负责输出逻辑，将Action Creator方法传递到组件的props中，在组件内即可发起Action。
示例代码如下：

``` js
import { connect } from 'react-redux';

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link);

export default FilterLink;
```


#### mapStateToProps
mapStateToProps是一个函数，接受两个参数，state和props，state包含整个全局对象树，props代表组件的props，一般情况下使用state情况最多。执行后返回一个对象，对象的键值对就是一个映射。
示例代码如下：


``` js
const mapStateToProps = (state, ownProps) => ({
  active: ownProps.filter === state.visibilityFilter
})
```


上述代码中，在mapStateToProps中，通过全局state和组件的props进行计算，得到包含键active的对象，传入组件，在组件中即可通过this.props.active来获取值。

#### mapDispatchToProps

mapDispatch是一个函数或者对象，用于将store.dispatch映射到组件的props。当mapDiapatchToProps是一个函数时，传入dispatch和ownProps两个参数，返回一个对象，对象的键值对定义了组件的props传递的方法以及对应的action。
示例代码如下：


``` js
const mapDispatchToProps = (
  dispatch,
  ownProps
) => {
  return {
    onClick: () => {
      dispatch({
        type: 'SET_VISIBILITY_FILTER',
        filter: ownProps.filter
      });
    }
  };
}
```

上述代码将dispatch方法映射到组件的props参数onClick，在组件内部即可通过this.props.onClick（）进行调用。

当mapDispatch是一个对象时，它的键值对分别是组件的props参数和一个作为Action Creator的函数，通常在大型应用中，会在专门的文件中定义actionCreators文件，如下：


``` js
export const setVisibilityFilter = (filter) => ({
  type: 'SET_VISIBILITY_FILTER',
  filter
})
```

上述代码就是一系列actionCreator之一，返回一个Action，由Redux自动发出，上述代码使用如下：

``` js
import { setVisibilityFilter } from '../actions';

const mapDispatchToProps = (dispatch, ownProps) => ({
  onClick: () => {
    dispatch(setVisibilityFilter(ownProps.filter))
  }
})
```

这种方法跟写成函数的方法其实是一样的，只是将action进行了单独的抽离，这样可以避免更多的冗余代码，如需要在多个组件中发起相同的action时。

### 实战To do应用
前面已经了解了react-redux的核心API，下面来完成一个完整的应用。

整个应用实现的界面如下：

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_183.png)

功能主要有，输入文字添加todo事项，显示todo list，点击todo项切换状态，点击tab筛选不同的todo 状态。

在应用开发过程中，使用components，首先，使用create-react-app来初始化react应用，src为主要的开发目录，src目录下，分别新建actions、components、containers、reducers文件夹，actions用于编写actionCreator，reducers文件夹下用于编写应用的reducer，component文件夹下编写UI组件，即无状态组件，只负责UI的呈现，不负责业务逻辑，数据通过props传入，也不操作redux相关的api。containers文件夹下编写容器组件，负责管理数据和业务逻辑，操作redux的api。

首先，在**App.js**中，包含了整个应用的UI界面。

``` js
import React, { Component } from 'react';
import Footer from './components/Footer';
import AddTodo from './containers/AddTodo';
import VisibleTodoList from './containers/VisibleTodoList';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {

    return (
      <div className="App">
        <div className="todoapp">
          <AddTodo/>
          <VisibleTodoList/>
          <Footer/>
        </div>
      </div>
    );
  }
}

export default App;
```


可以看到，整个应用包含三个组件**AddTodo**、**VisibleTodoList**、**Footer**。

下面逐步介绍整个界面的实现。
从containers文件夹开始，首先是AddTodo组件，用于添加todo。

``` js
import React from 'react'
import { connect } from 'react-redux'
import { addTodo } from '../actions'

let AddTodo = ({ dispatch }) => {
  let input

  return (
    <div>
      <form onSubmit={e => {
        e.preventDefault()
        if (!input.value.trim()) {
          return
        }
        dispatch(addTodo(input.value))
        input.value = ''
      }}>
        <input ref={node => {
          input = node
        }} />
        <button type="submit">
          Add Todo
        </button>
      </form>
    </div>
  )
}
AddTodo = connect()(AddTodo)

export default AddTodo
```


AddTodo包含一个Form表单，在Form表单的onSubmit事件中，获取输入值，提交至action，整个组件被redux的connect组件包裹，从而使得dispatch传入到组件的props中。

下面是actions的addTodo方法。

``` js
let nextTodoId = 0
export const addTodo = (text) => ({
  type: 'ADD_TODO',
  id: nextTodoId++,
  text
})
```

addTodo用于新建Todo，将todo的id加1,传入todo的text属性。

下面是VisibleTodoList组件，根据当前选中的tab来显示不同状态下的todo list。

``` js
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'

const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
    default:
      throw new Error('Unknown filter: ' + filter)
  }
}

const mapStateToProps = (state) => ({
  todos: getVisibleTodos(state.todos, state.visibilityFilter)
})

const mapDispatchToProps = {
  onTodoClick: toggleTodo
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```


从代码中可以看到，VisibleTodoList没有界面相关的部分，仅仅是将TodoList组件进行包裹返回一个新的组件。mapStateToProps中，根据当前所有的todos和筛选条件visibilityFilter，通过getVisibleTodos函数，返回一个筛选后的todos，mapDispatchToProps则是将actionCreator方法通过props传入到UI组件。


在actions文件夹下，index.js中，定义了toggleTodo如下：


``` js
export const toggleTodo = (id) => ({
  type: 'TOGGLE_TODO',
  id
})
```


toggleTodo通过传入的id对当前操作的todo进行处理。


下面是components下的TodoList组件，负责显示todo list。


``` js
import React from 'react'
import PropTypes from 'prop-types'
import Todo from './Todo'

const TodoList = ({ todos, onTodoClick }) => (
  <ul>
    {todos.map(todo =>
      <Todo
        key={todo.id}
        {...todo}
        onClick={() => onTodoClick(todo.id)}
      />
    )}
  </ul>
)

TodoList.propTypes = {
  todos: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number.isRequired,
    completed: PropTypes.bool.isRequired,
    text: PropTypes.string.isRequired
  }).isRequired).isRequired,
  onTodoClick: PropTypes.func.isRequired
}

export default TodoList
```


从上述代码可以看到，TodoList接收props中的todos, onTodoClick，并对todos进行渲染，每个todo又抽离成单独的Todo组件，在每个Todo上传递点击事件，点击事件中，调用onTodoClick，并传入当前todo的id，以此来改变todo的状态。

Todo组件如下所示：

``` js
import React from 'react'
import PropTypes from 'prop-types'

const Todo = ({ onClick, completed, text }) => (
  <li
    onClick={onClick}
    style={{
      textDecoration: completed ? 'line-through' : 'none'
    }}
  >
    {text}
  </li>
)

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  completed: PropTypes.bool.isRequired,
  text: PropTypes.string.isRequired
}

export default Todo
```

在TodoList组件中，渲染所有的todo list时，将每个todo进行对象解构，传入Todo组件的props，在Todo组件中，获取到组件需要的参数onClick, completed, text，onClick用于绑定到todo的点击事件，completed用于表示todo的状态是否为完成，根据completed的值来渲染todo的样式，text渲染为todo的内容。
下面是components中的Footer组件：

``` js
import React from 'react'
import FilterLink from '../containers/FilterLink'

const Footer = () => (
  <p>
    Show:
    {" "}
    <FilterLink filter="SHOW_ALL">
      All
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_ACTIVE">
      Active
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_COMPLETED">
      Completed
    </FilterLink>
  </p>
)

export default Footer
```

Footer组件中，用于对todos的状态进行筛选，共包含三个筛选项，SHOW_ALL显示所有todos，SHOW_ACTIVE显示刚添加的todo，SHOW_COMPLETED显示已经完成的todo。每个筛选项又抽离成FilterLink组件，传入filter作为筛选参数。

下面是container中FilterLink的代码。


``` js
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../actions'
import Link from '../components/Link'

const mapStateToProps = (state, ownProps) => ({
  active: ownProps.filter === state.visibilityFilter
})

const mapDispatchToProps = (dispatch, ownProps) => ({
  onClick: () => {
    dispatch(setVisibilityFilter(ownProps.filter))
  }
})

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)

export default FilterLink
```


FilterLink组件也仅仅是负责数据的逻辑，没有UI呈现。mapStateToProps中判断当前组件的props.filter与筛选条件state.visibilityFilter来计算active属性传入props，mapDispatchToProps定义onClick方法，用于dispatch一个action来改变当前的筛选条件，即state.visibilityFilter，传入当前组件的filter属性作为参数。

actions的setVisibilityFilter代码如下：

``` js
export const setVisibilityFilter = (filter) => ({
  type: 'SET_VISIBILITY_FILTER',
  filter
})
```


下面是Link组件。


``` js
import React from 'react'
import PropTypes from 'prop-types'

const Link = ({ active, children, onClick }) => {
  if (active) {
    return <span>{children}</span>
  }

  return (
    // eslint-disable-next-line
    <a href="#"
       onClick={e => {
         e.preventDefault()
         onClick()
       }}
    >
      {children}
    </a>
  )
}

Link.propTypes = {
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func.isRequired
}

export default Link
```


Link组件通过props接收active，children，onClick属性值，当active为true时，Link渲染为span，否则渲染为a，a标签绑定点击事件，点击时，触发props中的onClick方法。

自此，界面部分的代码都已经介绍完毕，下面是reducer的编写。
在reducer的编写中，为了让reducer的职责更为清晰，将reducer拆分为todos和visibilityFilter，todos.js中，用于处理todo的添加，状态改变。代码如下：

``` js
const todos = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          id: action.id,
          text: action.text,
          completed: false
        }
      ]
    case 'TOGGLE_TODO':
      return state.map(todo =>
        (todo.id === action.id)
          ? {...todo, completed: !todo.completed}
          : todo
      )
    default:
      return state
  }
}

export default todos
```


此处，todos初始化为一个空数组，reducer必须为纯函数，处理actions，不能直接对state进行修改，应该每次都返回一个新的state，当添加Todo时，使用数组展开运算符对state进行解构，并且，将新的todo对象作为数组的最后一个值，最终，生成一个全新的state。当修改todo状态时，使用数组的map方法，同样返回一个新的state，在map方法内部，使用三元运算符对action参数id和数组每一项的id进行判断，目的是寻找到当前被点击时的todo，将其completed值取反，否则直接返回当前todo。

visibilityFilter.js中，主要是用于更改todo list的筛选条件，代码如下：


``` js
const visibilityFilter = (state = 'SHOW_ALL', action) => {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter
    default:
      return state
  }
}

export default visibilityFilter

```

此时，state初始化为SHOW_ALL，即应用加载时，默认显示为全部的todos，当处理actions，直接返回action.filter。

当reducer被拆分为单个文件时，上述const 命名的函数名即为全局状态树中的state值，还需要将拆分的reducer进行组合，组合reducer主要使用了redux的combineReducers API，以下为reducers文件夹下index.js代码：

``` js
import { combineReducers } from 'redux'
import todos from './todos'
import visibilityFilter from './visibilityFilter'

const todoApp = combineReducers({
  todos,
  visibilityFilter
})

export default todoApp
```


最后一步，需要使用react-redux的Provider API包裹整个应用，src下index.js中：

``` js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux'
import './index.css';
import App from './App';
import { createStore } from 'redux';
import reducer from './reducers';

const store = createStore(reducer);

ReactDOM.render(
<Provider store={store}>
<App/>
</Provider>,
document.getElementById('root')
);
```
浏览器运行效果如下：

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_209.png)

至此，整个To do应用就开发完毕了，对比使用redux开发，react-redux让我们开发react应用的时候更加简单。

