---
title: Webpack之打包体积优化
date: 2018-10-30 22:39:51
banner: http://img.yanyuanfe.cn/webpack.png
tags:
- Webpack
---
> 本文是我在使用Webpack4过程中的一些总结，介绍一些优化Webpack打包体积的方法。

![image](http://img.yanyuanfe.cn/webpack.png)

<!--more-->
Webpack是当前大型项目的主流打包方案，自从将react的脚手架升级Webpack4以来，我也在不断尝试摸索webpack打包优化的一些方案。本文将从以下几个方面，对于如何优化Webpack的打包体积，进行一些总结。


### Bundle分析

webpack-bundle-analyzer可以对打包输出的chunk进行可视化分析, 可以看到打包后每个模块的大小，还能给出gizp压缩后的大小，在生产环境中加载的模块都是经过gzip压缩过的，可以作为真实访问的大小依据。

``` bash
npm install --save-dev webpack-bundle-analyzer
```

``` js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin()
  ]
}
```



### Tree-Shaking

Webpack4在mode为production的情况下默认开启了Tree-Shaking，但在你的项目中可能会因为babel的原因导致它失效。

因为Tree Shaking这个功能是依赖于 ES2015 模块系统中的静态结构特性，例如 import 和 export。来找出未使用的代码，所以如果你使用了babel插件的时候，如：babel-preset-env，它默认会将模块打包成commonjs，这样就会让Tree Shaking失效了。

所以需要配置 babel 不transform modules。配置如下：

``` js
// .babelrc
{
  "presets": [
    ["env", {
      modules: false,
      ...
    }]
  ]
}
```

### url-loader、file-loader配置优化

url-loader和file-loader一般用来处理背景图，默认会把css中的背景图打包成Base64，当css中的背景图过大，会造成打包后的css变得庞大，在单页应用中，css阻塞DOM渲染，造成首页白屏时间过长。曾遇到过某项目build后css多达4M，严重影响用户体验，解决办法是配置url-loader的limit字段，超过一定字节则不打包成base64。配置后重新打包后的css约300k。


``` js
{
    test: /\.(jpg|png|gif|svg)$/,
    loader: 'url-loader',
    include: path.join(__dirname, './src'),
    exclude: /node_modules/,
    options: {
        limit: 8192,
        name: 'images/[name].[hash:7].[ext]'
    }
}
```
### 按需加载
当我们使用React、Vue开发SPA应用时，webpack默认会输出一个js文件，这意味这首屏渲染的时候，会加载完所有的页面js，js体积过大极大影响首屏渲染速度，按需加载是一个不错的选择，对每一个路由对应的页面打包成单独的chunk。目前最流行的办法是使用动态import的方法。在react项目中，你可以使用react-loadable这个第三方库。


``` js
import Loadable from 'react-loadable';
import Loading from './my-loading-component';

const LoadableComponent = Loadable({
  loader: () => import('./my-component'),
  loading: Loading,
});

<Route path="/user" component={LoadableComponent} />
```
webpack会自动拆分chunk，首屏渲染时，只会加载对应的chunk，提高了渲染速度。

### 按需打包mement、lodash、antd
moment、lodash、antd是使用频率很高的第三方库，但是库本身的体积很大，项目中又不会用到全部的功能，全部引入会造成打包后的体积过大。

antd可以使用babel-plugin-import来进行按需加载，只需要安装babel-plugin-import然后配置.babelrc文件：


``` js
{
  "plugins": [
    [
      "import", {
        "libraryName": "antd",
        "style": true
      }
    ]
  ]
}

```
babel-plugin-import目前已经可以使用于antd、antd-mobile、lodash等库。
lodash也可以使用babel-plugin-import来进行按需加载，也可以使用babel-plugin-lodash来按需加载lodash。


``` bash
 npm i --save lodash
 npm i --save-dev babel-plugin-lodash @babel/cli @babel/preset-env
```


``` js
// .babelrc
{
  "plugins": ["lodash"],
  "presets": [["@babel/env", { "targets": { "node": 6 } }]]
}
```

moment会将所有本地化内容和核心功能一起打包，打包出来的 moment 依赖包括了许多不需要的 locale 文件。因此我们需要对 moment 的 locale 文件进行按需打包。

你可以使用 IgnorePlugin 在打包时忽略本地化内容:

``` js
new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/);
```

也可以使用webpack 的上下文替换插件(ContextReplacementPlugin)，允许覆盖查找规则，通过正则筛选指定的文件。

只需要在 webpack 的配置文件 plugins 处增加如下代码即可:


``` js
plugins: [
  new webpack.ContextReplacementPlugin(
    /moment[/\\]locale$/,
    /zh-cn/,
  ),
],
```

这里通过正则匹配了 moment/locale 下的名为 zh-cn 的文件。



