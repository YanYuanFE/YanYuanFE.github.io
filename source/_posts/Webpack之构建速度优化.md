---
title: Webpack之构建速度优化
date: 2018-10-29 21:33:46
banner: http://img.yanyuanfe.cn/webpack.png
tags:
- Webpack
---
> 本文是我在使用Webpack4过程中的一些总结，介绍一些优化Webpack构建速度的方法。

![image](http://img.yanyuanfe.cn/webpack.png)

<!--more-->

Webpack是当前最流行的打包工具，当前的最新版本是Webpack4+，性能有了很大的提升，本文是我在使用过程中的一些总结，主要是提高Webpack的构建性能。

### 保持版本更新
使用最新的 webpack 版本。新版本一般会进行性能优化。

保持最新的 Node.js 版本也能够提升性能。除此之外，保证你的包管理工具 (例如 npm 或者 yarn ) 为最新也能提升性能。较新的版本能够建立更高效的模块依赖树以及提高解析速度。

### 在Loader中使用include或者exclude
配置loader的时候，使用include，可以更精确指定要处理的目录，可以减少不必要的遍历，从而减少性能损失。同样，对于已经明确知道的，不需要处理的目录，则应该予以排除，从而进一步提升性能。故而，合理的设置include和exclude，将会极大地提升Webpack 打包优化速度。

``` js
module.exports = {
  //...
  module: {
    rules: [
      {
        test: /\.js$/,
        include: path.resolve(__dirname, 'src'),
        loader: 'babel-loader'
      }
    ]
  }
};
```

### 提高解析速度 
以下几步可以提高解析速度:

- 尽量减少 resolve.modules, resolve.extensions, resolve.mainFiles, resolve.descriptionFiles 中类目的数量，因为他们会增加文件系统调用的次数。
- 如果你不使用 symlinks ，可以设置 resolve.symlinks: false (例如 npm link 或者 yarn link).
- 如果你使用自定义解析 plugins ，并且没有指定 context 信息，可以设置 resolve.cacheWithContext: false 。

### module.noParse
防止 webpack 解析那些任何与给定正则表达式相匹配的文件。忽略的文件中不应该含有 import, require, define 的调用，或任何其他导入机制。忽略大型的类库 可以提高构建性能。

``` js
module.exports = {
  //...
  module: {
    noParse: /jquery|lodash/,

    // 从 webpack 3.0.0 开始
    noParse: function(content) {
      return /jquery|lodash/.test(content);
    }
  }
};
```


### babel-loader
babel-loader构建很慢，不仅要使用exclude、include，尽可能准确的指定要编译内容的目录，而且要充分利用缓存，进一步提升性能。babel-loader 提供了 cacheDirectory特定选项（默认 false），当有设置时，指定的目录将用来缓存 loader 的执行结果。之后的 webpack 构建，将会尝试读取缓存，来避免在每次执行时，可能产生的、高性能消耗的 Babel 重新编译过程。

通过使用 cacheDirectory 选项，将 babel-loader 提速至少两倍。 这会将转译的结果缓存到文件系统中。

babel 在每个文件都插入了辅助代码，使代码体积过大！ 
babel 对一些公共方法使用了非常小的辅助代码，比如 _extend。 默认情况下会被添加到每一个需要它的文件中

你可以引入 babel runtime 作为一个独立模块，来避免重复引入。

下面的配置禁用了 babel 自动对每个文件的 runtime 注入，而是引入 babel-plugin-transform-runtime 并且使所有辅助代码从这里引用。


``` js
rules: [
  // 'transform-runtime' 插件告诉 babel 要引用 runtime 来代替注入。
  {
    test: /\.js$/,
    exclude: /(node_modules|bower_components)/,
    use: {
      loader: 'babel-loader',
      options: {
        presets: ['@babel/preset-env'],
        plugins: ['@babel/transform-runtime']
      }
    }
  }
]
```
### Dlls

使用 DllPlugin 将更改不频繁的代码进行单独编译。这将改善引用程序的编译速度，即使它增加了构建过程的复杂性。

### Happypack多进程构建

由于Node.js的单进程限制，所有的loader虽然以async的形式来并发调用，但是还是运行在单个node的进程，以及在同一个事件循环中，这就直接导致了些问题：当同时读取多个loader文件资源时，比如babel-loader需要transform各种jsx，es6的资源文件。在这种CPU密集型的场景下，Node的单进程模型就没有优势了，而Happypack就是为解决此类问题而生。

Happypack 的处理思路是：将原有的 webpack 对 loader 的执行过程，从单一进程的形式扩展多进程模式，从而加速代码构建；原本的流程保持不变，这样可以在不修改原有配置的基础上，来完成对编译过程的优化，具体配置如下：


``` js
// webpack.config.js
const HappyPack = require('happypack');
const happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length });

exports.module = {
  rules: [
    {
      test: /.js$/,
      include: [resolve('src')],
      exclude: /node_modules/,
      loader: 'happypack/loader?id=happybabel'
    }
  ]
};

exports.plugins = [
  new HappyPack({
    id: 'happybabel',
    loaders: ['babel-loader'],
    threadPool: happyThreadPool,
    cache: true,
    verbose: true
  })
];
```
Happypack 在编译过程中，除了利用多进程的模式加速编译，还同时开启了 cache 计算，能充分利用缓存读取构建文件，对构建的速度提升也是非常明显的


