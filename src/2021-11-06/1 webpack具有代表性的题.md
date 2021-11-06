#  webpack 具有代表性的题


##### 问：Webpack 配置中用过哪些 Loader ？都有什么作用？

##### 问：Webpack 配置中用过哪些 Plugin ？都有什么作用？

##### 问：Loader 和 Plugin 有什么区别？

##### 问：如何编写 Loader ? 介绍一下思路？


##### 问：如何编写 Plugin ? 介绍一下思路？

##### 问：Webpack optimize 有配置过吗？可以简单说说吗？

##### 问：Webpack 层面如何性能优化？

##### 问：Webpack 打包流程是怎样的？


##### 问：tree-shaking 实现原理是怎样的？

##### 问：Webpack 热更新（HMR）是如何实现？


##### 问：Webpack 打包中 Babel 插件是如何工作的？

##### 问：Webpack 和 Rollup 有什么相同点与不同点？

##### 问：Webpack5 更新了哪些新特性？


webpack 知识体系图

![webpack 知识体系图](./images/webpack知识体系图.png "webpack 知识体系图")


围绕知识体系，简单分为三个层级：

- 基础 -- 会配置
- 进阶 -- 能优化
- 深入 -- 懂原理

## 一、Webpack 基础

- Webpack 常规配置项有哪些？
- 常用 Loader 有哪些？如何配置？
- 常用插件（Plugin）有哪些？如何的配置？
- Babel 的如何配置？Babel 插件如何使用？

> webpack 在 4 以后就支持 0 配置打包

##### 环境区分

**本地环境**

- 需要更快的构建速度
- 需要打印 debug 信息
- 需要 live reload 或 hot reload 功能
- 需要 sourcemap 方便定位问题

**生产环境**

- 需要更小的包体积，代码压缩+tree-shaking
- 需要进行代码分割
- 需要压缩图片体积

使用 cross-env 改变环境

```
"scripts": {
    "dev": "cross-env NODE_ENV=dev webpack serve --mode development", 
    "test": "cross-env NODE_ENV=test webpack --mode production",
    "build": "cross-env NODE_ENV=prod webpack --mode production"
  }
```

配置： 

```
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

console.log('process.env.NODE_ENV=', process.env.NODE_ENV) // 打印环境变量

const config = {
  entry: './src/index.js', // 打包入口地址
  output: {
    filename: 'bundle.js', // 输出文件名
    path: path.join(__dirname, 'dist') // 输出文件目录
  },
  module: { 
    rules: [
      {
        test: /\.css$/, //匹配所有的 css 文件
        use: 'css-loader' // use: 对应的 Loader 名称
      }
    ]
  },
  plugins:[ // 配置插件
    new HtmlWebpackPlugin({
      template: './src/index.html'
    })
  ]
}

module.exports = (env, argv) => {
  console.log('argv.mode=',argv.mode) // 打印 mode(模式) 值
  // 这里可以通过不同的模式修改 config 配置
  return config;
}
```

执行命令后:

```
process.env.NODE_ENV= prod
argv.mode= production
```


##### 配置本地环境

```
// webpack.config.js
const config = {
  // ...
  devServer: {
    contentBase: path.resolve(__dirname, 'public'), // 静态文件目录
    compress: true, //是否启动压缩 gzip
    port: 8080, // 端口号
    // open:true  // 是否自动打开浏览器
  },
 // ...
}
module.exports = (env, argv) => {
  console.log('argv.mode=',argv.mode) // 打印 mode(模式) 值
  // 这里可以通过不同的模式修改 config 配置
  return config;
}
```

**为什么要配置 contentBase ？**

因为 webpack 在进行打包的时候，对静态文件的处理，例如图片，都是直接 copy 到 dist 目录下面。但是对于本地开发来说，这个过程太费时，也没有必要，所以在设置 contentBase 之后，就直接到对应的静态目录下面去读取文件，而不需对文件做任何移动，节省了时间和性能开销。

> 注意：本文使用的 webpack-dev-server 版本是 3.11.2，当版本 version >= 4.0.0 时，需要使用 devServer.static 进行配置，不再有 devServer.contentBase 配置项。


##### 三种 hash 值


- ext	文件后缀名
- name	文件名
- path	文件相对路径
- folder	文件所在文件夹
- hash	每次构建生成的唯一 hash 值
- chunkhash	根据 chunk 生成 hash 值
- contenthash	根据文件内容生成hash 值

区别：

- hash ：任何一个文件改动，整个项目的构建 hash 值都会改变；
- chunkhash：文件的改动只会影响其所在 chunk 的 hash 值；
- contenthash：每个文件都有单独的 hash 值，文件的改动只会影响自身的 hash 值；


## 二、Webpack 进阶

### 1. 优化构建速度

使用插件 speed-measure-webpack-plugin

```
// 费时分析
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
const smp = new SpeedMeasurePlugin();
...

const config = {...}

module.exports = (env, argv) => {
  // 这里可以通过不同的模式修改 config 配置


  return smp.wrap(config);
}

```

> 注意：在 webpack5.x 中为了使用费时分析去对插件进行降级或者修改配置写法是非常不划算的，这里因为演示需要，我后面会继续使用，但是在平时开发中，建议还是不要使用。


### 1.2 优化 resolve 配置

#### 1.2.1 alias

alias 用的创建 import 或 require 的别名，用来简化模块引用，项目中基本都需要进行配置。

```
const path = require('path')
// 路径处理方法
function resolve(dir){
  return path.join(__dirname, dir);
}

 const config  = {
  ...
  resolve:{
    // 配置别名
    alias: {
      '~': resolve('src'),
      '@': resolve('src'),
      'components': resolve('src/components'),
    }
  }
};
```

#### 1.2.2 extensions

```
const config = {
  //...
  resolve: {
    extensions: ['.js', '.json', '.wasm'],
  },
};

```

那么 webpack 就会按照 extensions 配置的数组从左到右的顺序去尝试解析模块


需要注意的是：

- 高频文件后缀名放前面；
- 手动配置后，默认配置会被覆盖

如果想保留默认配置，可以用 ... 扩展运算符代表默认配置，例如

```
const config = {
  //...
  resolve: {
    extensions: ['.ts', '...'], 
  },
};
```

#### 1.2.3 modules


告诉 webpack 解析模块时应该搜索的目录，常见配置如下

```
const path = require('path');

// 路径处理方法
function resolve(dir){
  return path.join(__dirname, dir);
}

const config = {
  //...
  resolve: {
     modules: [resolve('src'), 'node_modules'],
  },
};

```
告诉 webpack 优先 src 目录下查找需要解析的文件，会大大节省查找时间

#### 1.2.4 resolveLoader

resolveLoader 与上面的 resolve 对象的属性集合相同， 但仅用于解析 webpack 的 loader 包。
一般情况下保持默认配置就可以了，但如果你有自定义的 Loader 就需要配置一下，不配可能会因为找不到 loader 报错

例如：我们在 loader 文件夹下面，放着我们自己写的 loader

我们就可以怎么配置

```
const path = require('path');

// 路径处理方法
function resolve(dir){
  return path.join(__dirname, dir);
}

const config = {
  //...
  resolveLoader: {
    modules: ['node_modules',resolve('loader')]
  },
};
```

#### 1.3 externals

externals 配置选项提供了「从输出的 bundle 中排除依赖」的方法。此功能通常对 library 开发人员来说是最有用的，然而也会有各种各样的应用程序用到它。

例如，从 CDN 引入 jQuery，而不是把它打包：

引入链接

```
<script
  src="https://code.jquery.com/jquery-3.1.0.js"
  integrity="sha256-slogkvB1K3VOkzAI8QITxV3VzpOnkeNVsKvtkYLMjfk="
  crossorigin="anonymous"
></script>
```
配置 externals

```
const config = {
  //...
  externals: {
    jquery: 'jQuery',
  },
};

```

使用 jQuery

```
import $ from 'jquery';

$('.my-element').animate(/* ... */);
```

#### 1.3 缩小范围

在配置 loader 的时候，我们需要更精确的去指定 loader 的作用目录或者需要排除的目录，通过使用 include 和 exclude 两个配置项，可以实现这个功能，常见的例如：

- include：符合条件的模块进行解析
- exclude：排除符合条件的模块，不解析
- exclude 优先级更高

```
const path = require('path');

// 路径处理方法
function resolve(dir){
  return path.join(__dirname, dir);
}

const config = {
  //...
  module: { 
    noParse: /jquery|lodash/,
    rules: [
      {
        test: /\.js$/i,
        include: resolve('src'),
        exclude: /node_modules/,
        use: [
          'babel-loader',
        ]
      },
      // ...
    ]
  }
};
```

#### 1.3 noParse


- 不需要解析依赖的第三方大型类库等，可以通过这个字段进行配置，以提高构建速度
- 使用 noParse 进行忽略的模块文件中不会解析 import、require 等语法

```
const config = {
  //...
  module: { 
    noParse: /jquery|lodash/,
    rules:[...]
  }

};
```

#### 1.4 IgnorePlugin

防止在 import 或 require 调用时，生成以下正则表达式匹配的模块：

requestRegExp 匹配(test)资源请求路径的正则表达式。
contextRegExp 匹配(test)资源上下文（目录）的正则表达式。

#### 1.5.1 thread-loader

配置在 thread-loader 之后的 loader 都会在一个单独的 worker 池（worker pool）中运行

```
// 路径处理方法
function resolve(dir){
  return path.join(__dirname, dir);
}

const config = {
  //...
  module: { 
    noParse: /jquery|lodash/,
    rules: [
      {
        test: /\.js$/i,
        include: resolve('src'),
        exclude: /node_modules/,
        use: [
          {
            loader: 'thread-loader', // 开启多进程打包
            options: {
              worker: 3,
            }
          },
          'babel-loader',
        ]
      },
      // ...
    ]
  }
};

```

#### 1.5.2 happypack

同样为开启多进程打包的工具，webpack5 已弃用。


#### 1.6 利用缓存
利用缓存可以大幅提升重复构建的速度
1.6.1 babel-loader 开启缓存

babel 在转译 js 过程中时间开销比价大，将 babel-loader 的执行结果缓存起来，重新打包的时候，直接读取缓存

缓存位置： node_modules/.cache/babel-loader

具体配置如下：

```
const config = {
 module: { 
    noParse: /jquery|lodash/,
    rules: [
      {
        test: /\.js$/i,
        include: resolve('src'),
        exclude: /node_modules/,
        use: [
          // ...
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true // 启用缓存
            }
          },
        ]
      },
      // ...
    ]
  }
}
```

那其他的 loader 如何将结果缓存呢？

**cache-loader** 就可以帮我们完成这件事情



#### 1.6.2 cache-loader

缓存一些性能开销比较大的 loader 的处理结果
缓存位置：node_modules/.cache/cache-loader

```
const config = {
 module: { 
    // ...
    rules: [
      {
        test: /\.(s[ac]|c)ss$/i, //匹配所有的 sass/scss/css 文件
        use: [
          // 'style-loader',
          MiniCssExtractPlugin.loader,
          'cache-loader', // 获取前面 loader 转换的结果
          'css-loader',
          'postcss-loader',
          'sass-loader', 
        ]
      }, 
      // ...
    ]
  }
}
```

#### 1.6.3 hard-source-webpack-plugin
hard-source-webpack-plugin 为模块提供了中间缓存，重复构建时间大约可以减少 80%，但是在 webpack5 中已经内置了模块缓存，不需要再使用此插件

#### 1.6.4 dll

在 webpack5.x 中已经不建议使用这种方式进行模块缓存，因为其已经内置了更好体验的 cache 方法

#### 1.6.5 cache 持久化缓存

通过配置 cache 缓存生成的 webpack 模块和 chunk，来改善构建速度。

```
const config = {
  cache: {
    type: 'filesystem',
  },
};

```

### 2. 优化构建结果

借助插件 webpack-bundle-analyzer 我们可以直观的看到打包结果中，文件的体积大小、各模块依赖关系、文件是够重复等问题，极大的方便我们在进行项目优化的时候，进行问题诊断。


```
// 引入插件
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin


const config = {
  // ...
  plugins:[ 
    // ...
    // 配置插件 
    new BundleAnalyzerPlugin({
      // analyzerMode: 'disabled',  // 不启动展示打包报告的http服务器
      // generateStatsFile: true, // 是否生成stats.json文件
    })
  ],
};

```

```
 "scripts": {
    // ...
    "analyzer": "cross-env NODE_ENV=prod webpack --progress --mode production"
  },
```

执行编译命令 npm run analyzer

打包结束后，会自行启动地址为 http://127.0.0.1:8888 的 web 服务，访问地址就可以看到

如果，我们只想保留数据不想启动 web 服务，这个时候，我们可以加上两个配置

```
new BundleAnalyzerPlugin({
   analyzerMode: 'disabled',  // 不启动展示打包报告的http服务器
   generateStatsFile: true, // 是否生成stats.json文件
})
```

#### 2.2 压缩 CSS

安装 optimize-css-assets-webpack-plugin

```
npm install -D optimize-css-assets-webpack-plugin 
```

```
// 压缩css
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin')
// ...

const config = {
  // ...
  optimization: {
    minimize: true,
    minimizer: [
      // 添加 css 压缩配置
      new OptimizeCssAssetsPlugin({}),
    ]
  },
 // 
}
```

#### 2.3 压缩 JS

> 在生成环境下打包默认会开启 js 压缩，但是**当我们手动配置 optimization 选项之后，就不再默认对 js 进行压缩，需要我们手动去配置**。

因为 webpack5 内置了terser-webpack-plugin 插件，所以我们不需重复安装，直接引用就可以了，具体配置如下

```
const TerserPlugin = require('terser-webpack-plugin');

const config = {
  // ...
  optimization: {
    minimize: true, // 开启最小化
    minimizer: [
      // ...
      new TerserPlugin({})
    ]
  },
  // ...
}
```
#### 2.4 清除无用的 CSS

purgecss-webpack-plugin 会单独提取 CSS 并清除用不到的 CSS

```
// ...
const PurgecssWebpackPlugin = require('purgecss-webpack-plugin')
const glob = require('glob'); // 文件匹配模式
// ...

function resolve(dir){
  return path.join(__dirname, dir);
}

const PATHS = {
  src: resolve('src')
}

const config = {
  plugins:[ // 配置插件
    // ...
    new PurgecssPlugin({
      paths: glob.sync(`${PATHS.src}/**/*`, {nodir: true})
    }),
  ]
}
```

#### 2.5 Tree-shaking

Tree-shaking 作用是剔除没有使用的代码，以降低包的体积

webpack 默认支持，需要在 .bablerc 里面设置 model：false，即可在生产环境下默认开启

了解更多 Tree-shaking 知识，推荐阅读 👉🏻 从过去到现在，聊聊 Tree-shaking

#### 2.6 Scope Hoisting

Scope Hoisting 即作用域提升，原理是将多个模块放在同一个作用域下，并重命名防止命名冲突，通过这种方式可以减少函数声明和内存开销。

- webpack 默认支持，在生产环境下默认开启
- 只支持 es6 代码

#### 3. 优化运行时体验

运行时优化的核心就是提升首屏的加载速度，主要的方式就是

降低首屏加载文件体积，首屏不需要的文件进行预加载或者按需加载

#### 3.1 入口点分割

配置多个打包入口，多页打包，这里不过多介绍

#### 3.2 splitChunks 分包配置

optimization.splitChunks 是基于 SplitChunksPlugin 插件实现的

默认情况下，它只会影响到按需加载的 chunks，因为修改 initial chunks 会影响到项目的 HTML 文件中的脚本标签。

webpack 将根据以下条件自动拆分 chunks：

- 新的 chunk 可以被共享，或者模块来自于 node_modules 文件夹
- 新的 chunk 体积大于 20kb（在进行 min+gz 之前的体积）
- 当按需加载 chunks 时，并行请求的最大数量小于或等于 30
- 当加载初始化页面时，并发请求的最大数量小于或等于 30

默认配置介绍

```
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async', // 有效值为 `all`，`async` 和 `initial`
      minSize: 20000, // 生成 chunk 的最小体积（≈ 20kb)
      minRemainingSize: 0, // 确保拆分后剩余的最小 chunk 体积超过限制来避免大小为零的模块
      minChunks: 1, // 拆分前必须共享模块的最小 chunks 数。
      maxAsyncRequests: 30, // 最大的按需(异步)加载次数
      maxInitialRequests: 30, // 打包后的入口文件加载时，还能同时加载js文件的数量（包括入口文件）
      enforceSizeThreshold: 50000,
      cacheGroups: { // 配置提取模块的方案
        defaultVendors: {
          test: /[\/]node_modules[\/]/,
          priority: -10,
          reuseExistingChunk: true,
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
      },
    },
  },
};

```

项目中的使用

```
const config = {
  //...
  optimization: {
    splitChunks: {
      cacheGroups: { // 配置提取模块的方案
        default: false,
        styles: {
            name: 'styles',
            test: /\.(s?css|less|sass)$/,
            chunks: 'all',
            enforce: true,
            priority: 10,
          },
          common: {
            name: 'chunk-common',
            chunks: 'all',
            minChunks: 2,
            maxInitialRequests: 5,
            minSize: 0,
            priority: 1,
            enforce: true,
            reuseExistingChunk: true,
          },
          vendors: {
            name: 'chunk-vendors',
            test: /[\\/]node_modules[\\/]/,
            chunks: 'all',
            priority: 2,
            enforce: true,
            reuseExistingChunk: true,
          },
         // ... 根据不同项目再细化拆分内容
      },
    },
  },
}

```

#### 3.3 代码懒加载

针对首屏加载不太需要的一些资源，我们可以通过懒加载的方式去实现，下面看一个小🌰

需求：点击图片给图片加一个描述

```js
// desc.js
const ele = document.createElement('div')
ele.innerHTML = '我是图片描述'
module.exports = ele
```

```js
// index.js
import './main.css';
import './sass.scss'
import logo from '../public/avatar.png'

import '@/fonts/iconfont.css'

const a = 'Hello ITEM'
console.log(a)

const img = new Image()
img.src = logo

document.getElementById('imgBox').appendChild(img)

// 按需加载
img.addEventListener('click', () => {
  import('./desc').then(({ default: element }) => {
    console.log(element)
    document.body.appendChild(element)
  })
})
```
#### 3.4 prefetch 与 preload

上面我们使用异步加载的方式引入图片的描述，但是如果需要异步加载的文件比较大时，在点击的时候去加载也会影响到我们的体验，这个时候我们就可以考虑使用 prefetch 来进行预拉取

##### 3.4.1 prefetch

- prefetch (预获取)：浏览器空闲的时候进行资源的拉取


```js
// 按需加载
img.addEventListener('click', () => {
  import( /* webpackPrefetch: true */ './desc').then(({ default: element }) => {
    console.log(element)
    document.body.appendChild(element)
  })
})
```
##### 3.4.2 preload


- preload (预加载)：提前加载后面会用到的关键资源
- ⚠️ 因为会提前拉取资源，如果不是特殊需要，谨慎使用

官网示例：

import(/* webpackPreload: true */ 'ChartingLibrary');


## 疑问

- external 和 noParse 有什么区别

作者：ITEM
链接：https://juejin.cn/post/7023242274876162084
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
