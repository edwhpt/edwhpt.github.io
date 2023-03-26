---
title: 【优化】Webpack构建性能
date: 2023-03-27 00:22:25
categories: Optimization
tags:
- Optimization
- Build
- Webpack
- Vite
---
当项目越来越复杂时，会面临着构建速度慢和构建出来的文件体积大的问题。webapck构建优化对于大项目是必须要考虑的一件事，下面就从速度和体积两方面来探讨构建优化的策略

<!-- more -->

## 分析工具

在优化之前，我们需要了解一些量化分析的工具，使用它们来帮助我们分析需要优化的点

### webpackbar

webpackbar可以在打包时实时显示打包进度

```js
const WebpackBar = require('webpackbar')
module.exports = {
  plugins: [new WebpackBar()]
}
```

### speed-measure-webpack-plugin

`speed-measure-webpack-plugin`可以看到每个loader和plugin的耗时情况

需要使用wrap方法包裹整个webpack配置项

```js
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin')
const smp = new SpeedMeasurePlugin()
module.exports = smp.wrap({
  plugins: [new WebpackBar()]
})
```

### webpack-bundle-analyzer

`webpack-bundle-analyzer`以可视化的方式让我们直观地看到打包的bundle中到底包含哪些模块内容，以及每一个模块的体积大小。可以根据这些信息去分析项目结构，调整打包配置，进行优化

构建完成后，默认会在`http://127.0.0.1:8888/`展示分析结果

```javascript
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
module.exports = {
  plugins: [new BundleAnalyzerPlugin()]
}
```

`webpack-bundle-analyzer`会计算出模块文件在三种情形下的大小：

- stat：文件未经过任何转换的原始大小
- parsed：文件经过转换后的输出大小（比如babel-loader转换ES6->ES5、UglifyJsPlugin压缩等等）
- gzip：parsed后的文件，经过Gzip压缩的大小 使用`speed-measure-webpack-plugin`和`webpack-bundle-analyzer`本身也会增加打包时间（`webpack-bundle-analyzer`特别耗时），所以建议这两个插件在开发分析时使用，而在生产环境去掉

## 优化构建速度

### thread-loader 多进程打包

把它放置在其它loader之前，放置在这个`thread-loader`之后的 loaders会运行在一个单独的worker池中

```js
// webpack.base.js
{
  test: /\.js$/,
  use: [
  	'thread-loader',
    'babel-loader'
  ]
}
```

### cache-loader 缓存资源

缓存资源，提高二次构建的速度，在一些性能开销较大的 loader 之前添加此`cache-loader`

```js
// webpack.base.js
{
  test: /\.js$/,
  use: [
    'cache-loader',
    'thread-loader',
    'babel-loader'
  ],
}
```

### hard-source-webpack-plugin 缓存模块

```js
const HardSourceWebpackPlugin = require("hard-source-webpack-plugin")
module.exports = {
  plugins:[new HardSourceWebpackPlugin()]
}
```

### 开启热更新

比如你修改了项目中某一个文件，会导致整个项目刷新，这非常耗时间。如果只刷新修改的这个模块，其他保持原状，那将大大提高修改代码的重新构建时间

```js
// webpack.dev.js

//引入webpack
const webpack = require('webpack');
//使用webpack提供的热更新插件
 plugins: [
   new webpack.HotModuleReplacementPlugin()
 ],
 //最后需要在我们的devserver中配置
 devServer: {
   hot: true
 }

```

### exclude & include

- `exclude`：不需要处理的文件
- `include`：需要处理的文件

合理设置这两个属性，可以大大提高构建速度

通常来说，loader会处理符合匹配规则的所有文件。比如babel-loader，会遍历项目中用到的所有js文件，对每个文件的代码进行编译转换。而node_modules里的js文件基本上都是转译好了的，不需要再次处理，所以我们用 include/exclude 来帮我们避免这种不必要的转译

```js
// webpack.base.js
{
  test: /\.js$/,
  //使用include来指定编译文件夹
  include: path.resolve(__dirname, '../src'),
  //使用exclude排除指定文件夹
  exclude: /node_modules/,
  use: [
    'babel-loader'
  ]
}
```

### 动态链接库

上面的babel-loader可以通过include/exclude，避免处理node_modules里的第三方库。

但如果将第三方库代码和业务代码都打包进一个bundle文件，那么处理这个bundle文件的插件，比如uglifyjs-webpack-plugin、terser-webpack-plugin等，就没办法不处理里面第三方库内容。

其实第三方库代码基本都是成熟的，不用作什么处理。因此，我们可以将项目的第三方库代码分离出来。

常见的处理方式有三种：

1. Externals
2. SplitChunks
3. DllPlugin

Externals可以避免处理第三方库，但每一个第三方库都得在html文档中增加一个script标签来引入，一个页面过多的js文件下载会影响网页性能，而且有时我们只使用第三方库中的一小部分功能，用script标签全量引入不太合理。

SplitChunks在每一次构建时都会重新构建第三方库，不能有效提升构建速度。

这里推荐使用DllPlugin和DLLReferencePlugin（配合使用），它们是webpack的内置插件。DllPlugin会将不频繁更新的第三方库单独打包，当这些第三方库版本没有变化时，就不需要重新构建。

使用方法：

1. 使用DllPlugin打包第三方库
2. 使用DLLReferencePlugin引用manifest.json，去关联第1步中已经打好的包

- 新建一个webpack配置文件`webpack.dll.js`用于打包第三方库（第1步）

  ```js
  const path = require('path')
  const webpack = require('webpack')
  
  module.exports = {
    mode: 'production',
    entry: {
      three: ['three', 'dat.gui']   // 第三方库数组
    },
    output: {
      filename: '[name].dll.js',    //[name]就是在entry
      path: path.resolve(__dirname, 'dist/lib'),
      library: '[name]'
    },
    plugins: [
      new webpack.DllPlugin({
        name: '[name]',
        path: path.resolve(__dirname, 'dist/lib/[name].json') //manifest.json的存放位置
      })
    ]
  }
  ```

  打包好后，在`dist`目录下增加了一个lib文件夹

- 在`webpack.base.js`做一下修改，去关联第1步中已经打好的包（第2步）

  ```js
  module.exports = {
    plugins:[
      //修改CleanWebpackPlugin配置
      new CleanWebpackPlugin({
        cleanOnceBeforeBuildPatterns: ['!lib/**'] //在每次清楚dist目录时，不清理lib文件夹的内容
      }),
      // dll相关配置
      new webpack.DllReferencePlugin({    
        // 将manifest字段配置成我们第1步中打包出来的json文件
        manifest: require('./dist/lib/three.json')  
      })
    ]
  }
  ```

不仅仅是第三方库，业务代码中的基础库也可以通过进行DllPlugin分离

### 构建区分环境

区分环境去构建是非常重要的，我们要明确知道，开发环境时我们需要哪些配置，不需要哪些配置；而最终打包生产环境时又需要哪些配置，不需要哪些配置：

- `开发环境`：去除代码压缩、gzip、体积分析等优化的配置，大大提高构建速度
- `生产环境`：需要代码压缩、gzip、体积分析等优化的配置，大大降低最终项目打包体积

### 合理配置hash

我们要保证，改过的文件需要更新hash值，而没改过的文件依然保持原本的hash值，这样才能保证在上线后，浏览器访问时没有改变的文件会命中缓存，从而达到性能优化的目的

```js
// webpack.base.js
  output: {
    path: path.resolve(__dirname, '../dist'),
    // 给js文件加上 contenthash
    filename: 'js/chunk-[contenthash].js',
    clean: true,
  },
```

### 提升webpack版本

webpack版本越新，打包的效果肯定更好



## 优化构建体积

主要是打包后项目整体体积的优化，有利于项目上线后的页面加载速度提升

### 代码分割

分离第三方库和业务代码中的基础库，可以避免单个bundle.js体积过大，加载时间过长。并且在多页面构建中，还能减少重复打包。

常见的操作是通过**SplitChunks**和 **动态链接库**（如上所示）

### 模块懒加载

如果不进行`模块懒加载`的话，最后整个项目代码都会被打包到一个js文件里，单个js文件体积非常大，那么当用户网页请求的时候，首屏加载时间会比较长，使用`模块懒加载`之后，大js文件会分成多个小js文件，网页加载时会按需加载，大大提升首屏加载速度

```js
// src/router/index.js
const routes = [
  {
    path: '/login',
    name: 'login',
    component: login
  },
  {
    path: '/home',
    name: 'home',
    // 懒加载
    component: () => import('../views/home/home.vue'),
  },
]
```

### 代码压缩

#### CSS代码压缩

CSS代码压缩使用`css-minimizer-webpack-plugin`，效果包括压缩、去重

```js
// webpack.prod.js
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')
module.exports = {
  optimization: {
    minimizer: [
      new CssMinimizerPlugin(), // 去重压缩css
    ],
  }
}
```

#### JS代码压缩

常用的js代码压缩插件有：`uglifyjs-webpack-plugin` 和 `terser-webpack-plugin`。

在webpack4中，生产环境默认开启代码压缩。我们也可以自己配置去覆盖默认配置，来完成更定制化的需求。

v4.26.0版本之前，webpack内置的压缩插件是uglifyjs-webpack-plugin，从v4.26.0版本开始，换成了`terser-webpack-plugin`。我们这里也以`terser-webpack-plugin`为例，和普通插件使用不同，在`optimization.minimizer`中配置压缩插件

使用`terser-webpack-plugin`，实现打包后JS代码的压缩

```js
// webpack.prod.js
const TerserPlugin = require('terser-webpack-plugin')
module.exports = {
  optimization: {
    minimizer: [
      new CssMinimizerPlugin(), // 去重压缩css
      new TerserPlugin({ // 压缩JS代码
        parallel: true,  // 开启并行压缩，可以加快构建速度
        sourceMap: true, // 如果生产环境使用source-maps，则必须设置为true
        terserOptions: {
          compress: {
            drop_console: true, // 去除console
          },
        },
      }), // 压缩JavaScript
    ],
  }
}  
```

### 图片处理

#### 雪碧图

雪碧图将多张小图标拼接成一张大图，在HTTP1.x环境下，雪碧图可以减少HTTP请求，加速网页的显示速度。

用于合成雪碧图的图标体积要小，较大的图片不建议拼接成雪碧图；同时要是网站静态图标，不是通过ajax请求动态获取的图标。所以通常是作为网站logo、icon之类的图片。

开发时，可以是UI提供雪碧图，但是每新增一个图标，就要重新制作一次，重新计算偏移量，比较麻烦。通过webpack插件合成雪碧图，就可以在开发时直接使用单个小图标，在打包时，自动合成雪碧图，并自动自动修改css中的`background-position`的值。

下面，我们借助`postcss-sprites`来自动合成雪碧图。

首先，在`webpack.base.js`中配置`postcss-loader`：

```javascript
//webpack.base.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['vue-style-loader','css-loader', 'postcss-loader']  //配置postcss-loader
      },
      {
        test: /\.less$/,
        use: [
          'vue-style-loader','css-loader', 'postcss-loader', 'less-loader']  //配置postcss-loader
      }
    ]
  }
};
```

然后在项目根目录下新建`.postcssrc.js`，配置`postcss-sprites`。

```javascript
module.exports = {
  "plugins": [
    require('postcss-sprites')({
      // 默认会合并css中用到的所有静态图片
      // 使用filterBy指定需要合并的图片，比如这里这里只合并images/icon文件夹下的图片
      filterBy: function (image) {
        if (image.url.indexOf('/images/icon/') > -1) {
            return Promise.resolve();
        }
        return Promise.reject();
      }
    })
  ]
}
```

默认会把图片合并到名为`sprite.png`的雪碧图中。

在css中直接指定小图标当背景：

```css
.star{
  display: inline-block;
  height: 100px;
  width: 100px;
  &.l1{
    background: url('../icon/star.png') no-repeat;
  }
  &.l2{
    background: url('../icon/star2.png') no-repeat;
  }
  &.l3{
    background: url('../icon/star3.png') no-repeat;
  }
}
```

打包完成后可以看到，自动修改了`background-image`和`background-position`。

#### 小图片转base64

对于一些小图片，可以转base64，这样可以减少用户的http网络请求次数，提高访问速度。`webpack5`中`url-loader`已被废弃，改用`asset-module`

```js
// webpack.base.js
{
   test: /\.(png|jpe?g|gif|svg|webp)$/,
   type: 'asset',
   parser: {
     // 转base64的条件
     dataUrlCondition: {
        maxSize: 25 * 1024, // 25kb
     }
   },
   generator: {
     // 打包到 image 文件下
    filename: 'images/[contenthash][ext][query]',
   },
}
```

### tree-shaking

`tree-shaking`简单说作用就是：只打包用到的代码，没用到的代码不打包，而`webpack5`默认开启`tree-shaking`，当打包的`mode`为`production`时，自动开启`tree-shaking`进行优化

```js
module.exports = {
  mode: 'production'
}
```

### source-map类型

`source-map`的作用是：方便你报错的时候能定位到错误代码的位置。它的体积不容小觑，所以对于不同环境设置不同的类型是很有必要的。

- **开发环境**

开发环境的时候我们需要能精准定位错误代码的位置

```js
// webpack.dev.js

module.exports = {
  mode: 'development',
  devtool: 'eval-cheap-module-source-map'
}
```

- **生产环境**

生产环境，我们想开启`source-map`，但是又不想体积太大，那么可以换一种类型

```js
// webpack.prod.js

module.exports = {
  mode: 'production',
  devtool: 'nosources-source-map'
}
```

### Gzip

开启gzip压缩，可以减小文件体积。在浏览器支持gzip的情况下，可以加快资源加载速度。服务端和客户端都可以完成gzip压缩，服务端响应请求时压缩，客户端应用构建时压缩。但压缩文件这个过程本身是需要耗费时间和CPU资源的，如果存在大量的压缩需求，会加大服务器的负担。

所以可以在构建打包时候就生成gzip压缩文件，作为静态资源放在服务器上，接收到请求后直接把压缩文件返回。

使用webpack生成gzip文件需要借助`compression-webpack-plugin`，使用配置如下：

```javascript
const CompressionWebpackPlugin = require("compression-webpack-plugin")
module.exports = {
  plugins: [
     new CompressionWebpackPlugin({
       test: /\.(js|css)$/,         //匹配要压缩的文件
       algorithm: "gzip"
     })
  ]
}
```



## 最后（对比Vite）

可以考虑使用Vite替换Webpack

[Vite](https://cn.vitejs.dev/) 是一种新型前端构建工具，能够显著提升前端开发体验。它主要由两部分组成：

- 一个开发服务器，它基于 [原生 ES 模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 提供了 [丰富的内建功能](https://cn.vitejs.dev/guide/features.html)，如速度快到惊人的 [模块热更新（HMR）](https://cn.vitejs.dev/guide/features.html#hot-module-replacement)。
- 一套构建指令，它使用 [Rollup](https://rollupjs.org/) 打包你的代码，并且它是预配置的，可输出用于生产环境的高度优化过的静态资源。

Vite 意在提供开箱即用的配置，同时它的 [插件 API](https://cn.vitejs.dev/guide/api-plugin.html) 和 [JavaScript API](https://cn.vitejs.dev/guide/api-javascript.html) 带来了高度的可扩展性，并有完整的类型支持。

你可以在 [为什么选 Vite](https://cn.vitejs.dev/guide/why.html) 中了解更多关于项目的设计初衷。

### Webpack和Vite的区别

- webpack会先打包，然后启动开发服务器，请求服务器时直接给予打包结果
- vite直接启动开发服务器，请求哪个模块再对该模块进行实时编译

### vite优点

- **webpack服务器启动速度比vite慢**
  由于vite启动的时候不需要打包，也就无需分析模块依赖、编译，所以启动速度非常快。当浏览器请求需要的模块时，再对模块进行编译，这种按需动态编译的模式，极大缩短了编译时间，当项目越大，文件越多时，vite的开发时优势越明显
- **vite热更新比webpack快**
  vite在HRM方面，当某个模块内容改变时，让浏览器去重新请求该模块即可，而不是像webpack重新将该模块的所有依赖重新编译；
- **vite使用esbuild(Go 编写) 预构建依赖，而webpack基于nodejs, 比node快 10-100 倍**

### vite缺点

- 生态不及webpack，加载器、插件不够丰富
- 打包到生产环境时，vite使用传统的rollup进行打包，生产环境esbuild构建对于css和代码分割不够友好。所以，vite的优势是体现在开发阶段
- 没被大规模重度使用，会隐藏一些问题
- 项目的开发浏览器要支持esmodule，而且不能识别commonjs语法
