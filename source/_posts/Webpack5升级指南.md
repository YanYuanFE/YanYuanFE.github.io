---
title: Webpack5升级指南
date: 2020-11-5 20:05:50
tags:
  - Webpack
---

> Webpack5最近发布了正式版，带来了很多优化和新功能，前端工程的构建效率将会大大提升，并且，相比于Webpack4，v5的升级也更加平滑，不会有很多破坏性的变更。

![image](http://img.yanyuanfe.cn/webpack.jpeg)

<!--more-->
Webpack5最近发布了正式版，带来了很多优化和新功能，前端工程的构建效率将会大大提升，并且，相比于Webpack4，v5的升级也更加平滑，不会有很多破坏性的变更。

本次重大发布的整体发展方向如下：

- 尝试用持久性缓存来提高构建性能。
- 尝试用更好的算法和默认值来改进长期缓存。
- 尝试用更好的 Tree Shaking
- 和代码生成来改善包大小。
- 尝试改善与网络平台的兼容性。
- 尝试在不引入任何破坏性变化的情况下，
- 清理那些在实现 v4 功能时处于奇怪状态的内部结构。
- 试图通过现在引入突破性的变化来为未来的功能做准备，
- 尽可能长时间地保持在 v5 版本上。

为了尝试新特性带来的变化，我对一些自己的开源项目进行了依赖升级，本文将对我升级过程中的一些问题的总结。

### 使用npm-check-updates进行依赖升级

npm-check-updates 插件会自动检查package.json里的最新版本，并进行批量升级。

```
npm install -g npm-check-updates

```


```
ncu
```
ncu命令会自动检查package.json中的依赖的最新版本，并列出可升级依赖。


```
ncu -u
```
运行此命令可对package.json中的依赖版本进行批量更新至最新版本号。

然后运行yarn或者npm install安装最新版本。

### Typescript 类型

Webpack4使用@types/webpack来进行类型检查， Webpack 5 从源码中生成 typescript 类型文件。

修改：


```
yarn remove @types/webpack
```


### webpack-cli

webpack4.x的启动项目和build命令使用：


```
"scripts": {
    "start": "webpack-dev-server --config ./config/webpack.config.dev.ts --progress --colors",
    "build": "webpack-cli --config ./config/webpack.config.prod.ts --progress --colors"
  },
```

运行npm start，出现如下报错：


```
Error: Cannot find module 'webpack-cli/bin/config-yargs' 
```
对应issue：https://github.com/webpack/webpack-dev-server/issues/2424

V5后，使用webpack-cli/serve来代替webpack-dev-server的启动命令：

对应的script修改为：


```
"scripts": {
    "start": "webpack-cli serve --config ./config/webpack.config.dev.ts",
    "build": "webpack-cli --config ./config/webpack.config.prod.ts --progress --colors"
  },
```

### 命令行参数

webpack-cli在webpack5的版本中删除和新增了一些命令行参数，运行时会出现如下报错：

--colors


```
[webpack-cli] Unknown argument: --colors

```
以上报错就是--colors参数在v5版本不支持了，需要去掉。

不同版本支持的参数参考官方Github：https://github.com/webpack/webpack-cli/tree/next/packages/webpack-cli#webpack-5


### webpack loader


使用 query 参数来设置loader的options会在启动webpack时报错如下：

```
[webpack-cli] Promise rejection: Error: Compiling RuleSet failed: Query arguments on 'loader' has been removed in favor of the 'options' property (at ruleSet[1].rules[5].loader: file-loader?name=images/[name].[hash:5].[ext])
[webpack-cli] Error: Compiling RuleSet failed: Query arguments on 'loader' has been removed in favor of the 'options' property (at ruleSet[1].rules[5].loader: file-loader?name=images/[name].[hash:5].[ext])

```


```
{
        test: /\.(png|jpg|jpeg|gif|svg)$/,
        loader: "file-loader",
        //loader: "file-loader?name=images/[name].[hash:5].[ext]",
        options: {
          name: 'images/[name].[hash:5].[ext]',
        },
      }
```

### devtool


```
webpack-cli] Invalid configuration object. Webpack has been initialized using a configuration object that does not match the API schema.
 - configuration.devtool should match pattern "^(inline-|hidden-|eval-)?(nosources-)?(cheap-(module-)?)?source-map$".
   BREAKING CHANGE since webpack 5: The devtool option is more strict.
   Please strictly follow the order of the keywords in the pattern.
```



### 移除Node.js模块的Polyfills

webpack4及其之前的版本附带了一些node模块的polyfill，如：cypto、buffer、process等，如果你的项目中使用到这些模块，则会自动应用相应的polyfill，V5版本中，移除了这些polyfill，如果需要使用则需要手动添加到配置文件：


```
plugins: [
    new ProvidePlugin({
      Buffer: ["buffer", "Buffer"],
      process: "process",
    }),
]
```

### 使用eslint-webpack-plugin代替eslint-loader

eslint-webpack-plugin的出现解决了一些eslint-loader的问题。
eslint-webpack-plugin提供了更好的配置方式，生成报告，直接从eslint使用缓存，仅仅lint修改过的文件。

总的来说，eslint-webpack-plugin更好用，首次启动速度大大提升。

修改：


```
yarn add eslint-webpack-plugin -D
```


```
const ESLintPlugin = require('eslint-webpack-plugin');

module.exports = {
  // ...
  plugins: [new ESLintPlugin({
      fix: true,
      lintDirtyModulesOnly: true,
    })],
  // ...
};
```

### babel-polyfill

在 Babel > 7.4.0 之前，通常我们会安装 babel-polyfill 或 @babel/polyfill来处理实例方法和ES+新增的内置函数，从Babel 7.4.0开始，不推荐使用此软件包，而通过直接导入core-js / stable（以充实ECMAScript功能）和regenerator-runtime / runtime（需要使用转译的生成器函数）：


```
import "core-js/stable";
import "regenerator-runtime/runtime";
```

下面直接介绍transform-runtime的方式：

安装依赖：
```
yarn add babel-loader @babel/core @babel/preset-env @babel/plugin-transform-runtime -D
yarn add @babel/runtime-corejs3
```
.babelrc配置文件：

```
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "modules": false,
      }
    ]
  ],
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "corejs": {
          "version": 3,
          "proposals": true
        },
        "useESModules": true
      }
    ]
  ]
}
```



参考：https://segmentfault.com/a/1190000020237817




参考：https://webpack.js.org/blog/2020-10-10-webpack-5-release/
