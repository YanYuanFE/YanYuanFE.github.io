---
title: "React Native 封装HTTP请求"
date: 2018-01-04 17:45:06
banner: http://img.yanyuanfe.cn/QQ20160705-3.png
tags:
 - React Native
---

> 作为一个前端开发者，我觉得React Native也是一个很酷的东西，本文不涉及React Native的开发相关，只是总结了开发中的一些最佳实践。不仅仅适合React Native。

![image](http://img.yanyuanfe.cn/react_1_1x.jpg)

<!--more-->

在进行项目开发时，我们都会将公共的模块抽离出来。在学习React Native的过程中，我也遇到了同样的场景，在此记录下来。React Native使用fetch作为网络请求库，而每个几乎每个模块都需要用到网络请求，那么将其抽离为公共模块甚为必要。
### 封装HTTP模块的优点
- 代码复用  

由于很多模块都会用到HTTP请求，将fetch封装为公共模块可以复用很多代码

- 高内聚低耦合

将公共部分提取出来，可以使每个模块专注于自身的业务逻辑

- 便于项目的扩展与后期的维护，并且做到职责分离


### 具体做法

首先，在项目根目录下新建utils目录，在utils目录中，新建fetch.js。

要想在其他模块中使用整个模块，首先，我们要将其导出为一个模块供其他模块使用。

``` js
export default class fetchUtils {
    
}
```
fetchUtils模块中应该包含两个方法，分别是get和post的请求方法，下一步，来编写get方法。


``` js
static get(url) {
    return new Promise((resolve, reject) => {
      fetch(url)
          .then(response => response.json())
          .then((result) => {
              resolve(result);
          })
          .catch(error => {
              reject(error);
          })

    })
}
```
get方法应该是一个静态方法，接受一个参数url，其返回一个Promise，Promise接受两个参数resolve和reject，用于处理成功和失败的情况，在Promise中，使用fetch发起get请求，然后使用链式函数then来处理服务器返回结果，先取出json数据，当服务器成功返回数据，使用resolve返回结果；当发生错误时，调用catch方法捕获错误。

下面是post方法。

``` js
static post(url, data) {
    return new Promise((resolve, reject) => {
        fech(url, {
            method: 'POST',
            header: {
                'Accept': 'application/json',
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(data)
        })
        .then(response => response.json())
        .then((result) => {
            resolve(result);
        })
        .catch(error => {
            reject(error);
        })
    })
}
```
post方法与get方法类似，也是静态方法，post方法接受两个参数，分别是url和提交的数据data。同样的，post方法返回一个Promise，参数与get方法一样，然后向服务器发送POST请求，post请求需要设置method为POST，还需要设置请求头，请求头包含接受的数据类型和Content-Type，均设置为application/json，最后将用户提交的数据放到body中，使用JSON序列化data。然后需要接受服务器返回的数据，这里与get方法一样，不再赘述。


以上就是封装好的HTTP请求模块。下面看一下如何使用。

首先，将fetchUtils导入。


```
import fetchUtils from '../utils/fetch'
```



**get方法**


``` js
fetchUtils.get(url)
    .then(result => {
        this.setState({result: JSON.stringify(result)})
    })
    .catch(error => {
         this.setState({result: JSON.stringify(error)})
    })
```

**post方法**


``` js
fetchUtils.post(url, data)
    .then(result => {
        this.setState({result: JSON.stringify(result)})
    })
    .catch(error => {
         this.setState({result: JSON.stringify(error)})
    })
```
### 总结

通过以上对fetch方法的封装，使代码更加精简，业务逻辑更加清晰，也更加容易维护。此处使用React Native为例只是项目开发的一个缩影，这种封装思想是可以在工作中借鉴的。

