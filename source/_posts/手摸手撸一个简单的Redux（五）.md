---
title: 手摸手撸一个简单的Redux（五）
date: 2018-04-04 21:30:00
banner: http://img.yanyuanfe.cn/photo-1432821596592-e2c18b78144f.jpeg
tags:
 - Redux
 - React
---

> Redux使用CombineReducer来组合多个reducer函数。

![image](http://img.yanyuanfe.cn/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67.png)

<!--more-->

之前我们已经完成了redux和react-redux的大部分功能，本文将结合之前的To do list项目来完善我们编写的redux和react-redux，主要是实现combineReducer以及mapDiapatch的默认参数和让mapDispatch支持function参数。

> 本文完整代码请查看Github：https://github.com/YanYuanFE/redux-app

``` bash
// clone repo
git clone https://github.com/YanYuanFE/redux-app.git


cd redux-app

// checkout branch
git checkout part-7

// install
npm install

// start
npm start

```

### combineReducer
随着应用变得越来越复杂，我们会将reducer根据业务进行拆分，拆分后的reducer函数负责独立管理state的一部分。

Redux为我们提供了combineReducer这个辅助函数，用于将我们拆分后的多个reducer根据自身的键值进行组合成一个新的object，成为新的reducer，然后对这个reducer调用createStore方法。

combineReducer合并后的reducer可以调用各个子reducer，并且把返回的结果合并成一个state对象。由combineReducer返回的state对象，会将传入的每个reducer返回的state按其传递给combineReducer时对应的key进行命名。

通常在项目中，我们会为每个单独的reducer单独创建js文件，在reducer中为每个reducer进行命名并导出，然后在reducer的入口文件中导入，通过Redux的combineReducer为每个reducer进行命名不同的key来控制不同的state的key的命名。
下面回顾一下在之前Todo list项目中对combineReducer的使用。
reducers/index.js：

``` js
import { combineReducers } from 'redux';
import todos from './todos';
import visibilityFilter from './visibilityFilter';

const todoApp = combineReducers({
  todos,
  visibilityFilter
});

export default todoApp;
```


combineReducers({todos, visibilityFilter})这里使用了ES6的对象简写语法，这与
combineReducers({todos: todos, visibilityFilter: visibilityFilter})是等价的。
此处需要注意，combineReducer中传入对象的key与redux中存储的state同名。

下面来实现一个简单的combineReducer。
在src/redux.js中：

``` js
export function combineReducers(reducers) {
  const finalReducerKeys = Object.keys(reducers);

  return (state = {}, action) => {
    let nextState = {};
    finalReducerKeys.forEach((key) => {
      const reducer = reducers[key];
      const prevStateForKey = state[key];
      const nextStateForKey = reducer(prevStateForKey, action);
      nextState[key] = nextStateForKey;
    });

    return nextState;
  }

}
```

可以看到，combineReducer是一个高阶函数，返回一个function，首先通过Object.keys获取到由reducers
的key组成的数组finalReducerKeys，然后返回一个function，这个function是一个组合后的reducer函数，
接收state和action参数，state默认为空对象，在function内部，定义nextState为空对象，然后对
finalReducerKeys进行遍历，通过数组的key获取到每一个reducer，然后为reducer传入前一个state和
action，返回新的state并加入到nextState的对象中，最后返回新的state。
我们还可以用更加简洁的代码来实现：

``` js
export function combineReducers(reducers) {
  
  const finalReducerKeys = Object.keys(reducers);

  return (state = {}, action) => {

    return finalReducerKeys.reduce((ret, item) => {
       ret[item] = reducers[item](state[item], action);
       return ret;
    }, {});
  }

}
```

上述代码中，使用reduce对一个空对象进行累加操作，对数组每一项进行计算并返回一个新的对象，代码更加简洁。


### 新的To do List

现在我们将之前使用react-redux实现的To do List项目使用自己实现的react-redux，修改containers文件夹下
的AddTodo.js、FilterLink.js、VisibleTodoList.js，将引用的react-redux修改为自己编写的
react-redux，如下：

``` js
import { connect } from '../react-redux'
```

然后运行项目，开始报错：

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_220.png)

emm。

好吧，查看报错信息，发现containers/FilterLink.js文件中：

``` js
import { connect } from '../react-redux'
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
)(Link);

export default FilterLink
```

在mapStateToProps和mapDispatchToProps都使用了ownProps参数，但是再看下我们自己的react-redux中
connect的参数。

``` js
update() {
  const { store } = this.context;
  const stateProps = mapStateToProps(store.getState());
  const dispatchProps = bindActionCreators(mapDispatchToProps, store.dispatch);

  this.setState({
    props: {
      ...this.state.props,
      ...stateProps,
      ...dispatchProps,
    }
  })
}
```

使用mapStateToProps并没有传入props，修改如下：

``` js
const stateProps = mapStateToProps(store.getState(), this.props);
```

然后，没有报错了，但是页面好像有点问题，下面的筛选按钮没有显示出来。

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_221.png)

使用React开发者工具查看，是因为props没有传递下去。修改上述代码，将ConectComponent
中的this.props也传递下去。

``` js
this.setState({
  props: {
    ...this.state.props,
    ...stateProps,
    ...dispatchProps,
    ...this.props,
  }
})
```

再查看界面，显示好了，下面尝试新增一个to do。输入提交后又报错了。

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_222.png)

查看报错信息：
``` js
TypeError: dispatch is not a function
```

原来是AddTodo组件的props中没有dispatch方法，分析如下，使用connect将AddTodo组件与Redux进行连接，
但是connect中并没有传递参数，mapDispatch参数被默认定义为空对象，这里应该默认定义为一个dispatch方法，
修改如下：

``` js
export const connect = (mapStateToProps = state => state, mapDispatchToProps) => (WrapComponent) => {
  return class ConectComponent extends React.Component {
    static contextTypes = {
      store: PropTypes.object
    }

    constructor(props, context) {
      super(props, context);
      this.state = {
        props: {}
      }
    }
    componentDidMount() {
      const { store } = this.context;
      store.subscribe(() => this.update());
      this.update();
    }
    update() {
      const { store } = this.context;
      const stateProps = mapStateToProps(store.getState(), this.props);
      if (!mapDispatchToProps) {
       
          mapDispatchToProps = { dispatch: store.dispatch}
      }
     

      this.setState({
        props: {
          ...this.state.props,
          ...stateProps,
          ...dispatchProps,
          ...this.props,
        }
      })
    }
    render() {
      return <WrapComponent {...this.state.props}/>
    }
  }
}
```

这里主要是去掉了mapDispatch的默认参数，在update函数中对其进行判断是否为空，为空则传递一个对象，对象包含
一个dispatch方法。
再次尝试添加todo，添加成功，但是出来了两条数据，可能是连续触发了两次dispatch，点击筛选按钮试试，报错了。

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_223.png)
查看报错信息：

``` js
TypeError: _onClick is not a function
```


components/Link.js中的onClick props没有传递进来，查看onClick方法定义的地方：
在containers/FilterLink.js中，在mapDispatch中进行了定义：

``` js
const mapDispatchToProps = (dispatch, ownProps) => ({
  onClick: () => {
    dispatch(setVisibilityFilter(ownProps.filter))
  }
})
```


在我们实现的connect中，mapDispatch只支持传递对象参数，下面进行修改，让其支持传递函数。
修改update方法：

``` js
let dispatchProps;
if (typeof mapDispatchToProps === 'function') {
    dispatchProps = mapDispatchToProps(store.dispatch, this.props);
} else {
    dispatchProps = bindActionCreators(mapDispatchToProps, store.dispatch);
}
```

上述代码的作用主要是对mapDispatch进行处理，判断其类型是否为function，如果为function则执行一下，
传入store.dispatch和this.props参数，返回一个对象，然后赋值到dispatchProps上。
然后尝试点击筛选按钮，没有报错，功能正常，但是添加todo还是会出现两条数据，经过调试发现是因为又执行了一
次bindActionCreator，尝试将mapDispatch的默认值修改为一个函数，如下：

``` js
if (!mapDispatchToProps) {
  mapDispatchToProps = (dispatch) => ({dispatch})
}
let dispatchProps;
if (typeof mapDispatchToProps === 'function') {
    dispatchProps = mapDispatchToProps(store.dispatch, this.props);
} else {
    dispatchProps = bindActionCreators(mapDispatchToProps, store.dispatch);
}
```

这样，下面对mapDispatch进行判断后，会返回一个对象，包含dispatch方法。
重新添加todo，成功，数据正确。如下图。
![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_224.png)

细心的你发现了吗？在右边的控制台一直会出现警告，大概意思是我们的组件props参数应该是一些数值但是实际上
是undefined，因为在组件内部定义了PropTypes。

``` js
Link.propTypes = {
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func.isRequired
}

export default Link
Link.propTypes = {
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func.isRequired
}

export default Link
```

问题应该出现在我们的connect方法中，在connect方法中，最终render返回包裹的组件，并传递this.state.props
到包裹的组件，但是this.state.props初始为空，只有在componentDidMount才会调用this.update方法更新state，
而组件执行是先render再执行componentDidMount，故第一次render时所有的props都是undefined，造成报错。
我们可以将componentDidMount修改为componentWillMount，componentWillMount在render之前执行，可以避免
这个报错。修改如下：

``` js
componentWillMount() {
  const { store } = this.context;
  store.subscribe(() => this.update());
  this.update();
}
```

最终效果如下，没有报错信息，尝试添加todo和修改，功能都正常。

![image](http://img.yanyuanfe.cn/%E9%80%89%E5%8C%BA_225.png)



参考：
> https://github.com/ecmadao/Coding-Guide/tree/master/Notes/React/Redux
