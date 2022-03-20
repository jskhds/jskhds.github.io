---
title: webpack性能优化
date: 2022-02-27 14:26:40
tags: webpack 性能优化
---

webpack 性能优化

<!--more-->

#### 1.优化打包效率

开发体验和效率

##### 插件

- ##### **优化 babel-loader**

```js
{
	test:/\.js$/,
	use: ['babel-loader?cacheDirectory'], // 开启缓存
	include: path.resolve(__dirname, 'src') // 明确打包范围
	// include 和 exclude 各选一个
	exclude: path.resolve(__dirname, 'node_modules')  // 明确不打包的范围
}
```



- ##### **IgnorePlugin** 

避免引用无用模块，具体步骤：先忽略 => 再动态引用

```js
// prod.js/plugins下 先忽略
new webpack.IgnorePlugin(/*正则匹配要忽略的文件*/, /*对应的插件*/)

// 在相应的文件下引入
import /*对应插件想要功能的目录*/

// eg moment 插件
new webpack.IgnorePlugin(/\./\locale/, moment)  // 忽略所有语言包
import moment from 'moment'
import 'moment/locale/zh-cn'  // 手动引入中文语言包 
moment.locale('zh-cn')   //把语言包设置成中文

```

- **noParse**

避免重复打包。webpack noParse作用主要是过滤不需要解析依赖的文件，比如打包的时候依赖了三方库（jquyer、lodash）等，而这些三方库里面没有其他依赖，可以通过配置noParse不去解析文件，提高打包效率

```js
module.exports = {
	module: {
        // 类似
		noParse: [/react\.min\.js$/, /jquery/],
	}
}
```

IgnorePlugin  是直接不引人，noParse 是引入但不打包

- **happyPack**

由于 JavaScript 是单线程的，要想发挥多核 CPU 的能力，只能通过多进程去实现，而无法通过多线程实现。happyPack 就可以多进程打包。

步骤：引入=>替换 loader => 创建插件

```js
const HappyPack = require('happypack');

module.exports = {
    //...
    module: {
        rules: [
            test: /\.js$/,
            // use: ['babel-loader?cacheDirectory'] before
            use: ['happypack/loader?id=babel'], // now
            exclude: /node_modules/             // 排除 node_modules 目录下的文件
        ]
    },
    plugins: [
        //...,
        new HappyPack({
            /*
             * 必须配置
             */
            // id 标识符，要和 rules 中指定的 id 对应起来
            id: 'babel',
            // 需要使用的 loader，用法和 rules 中 Loader 配置一样
            // 可以直接是字符串，也可以是对象形式
            loaders: ['babel-loader?cacheDirectory']
        })
    ]
}

```

- **ParallelUglifyPlugin**

多进程压缩 JS。因为 JS 是单线程的，所以开启多进程压缩会更快。原理同 happyPack

步骤：引用 => plugins 配置

```js
// 使用 ParallelUglifyPlugin 并行压缩输出的 JS 代码
   new ParallelUglifyPlugin({
       // 传递给 UglifyJS 的参数
       // （还是使用 UglifyJS 压缩，只不过帮助开启了多进程）
       uglifyJS: {
           output: {
               beautify: false, // 最紧凑的输出
               comments: false, // 删除所有的注释
           },
           compress: {
               // 删除所有的 `console` 语句，可以兼容ie浏览器
               drop_console: true,
               // 内嵌定义了但是只用到一次的变量
               collapse_vars: true,
               // 提取出出现多次但是没有定义成变量去引用的静态值
               reduce_vars: true,
           }
       }
   })
```



- **自动刷新**

一般来说用不上，开发环境下可能会用。devServer 配置一般会自动带上自动刷新 

```js
module.export = {
  watch: true,  //默认为 false 开启监听后 webpack-dev-server 会自动开启刷新浏览器
  watchOptions: {
    ignored: /node_modules/,
    // 监听到变化发生后会等300ms再去执行动作，防止文件更新太快导致重新编译频率太高  
    aggregateTimeout: 300,  
    // 判断文件是否发生变化是通过不停的去询问系统指定文件有没有变化实现的
    poll: 1000
  }
}

```

- **热更新**

自动刷新的缺点：整个网页全部刷新，速度较慢；整个网页刷新后状态丢失

热更新：新代码生效，网页不刷新，状态不丢失

步骤：安装 => 配置 => 按需引用 

```js
npm i webpack webpack-cli -D
npm i webpack-dev-server -D
npm i html-webpack-plugin -D

//配置 config 文件
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
  ...
  devServer: {
    hot: true, // 开启热更新
    port: 8080, // 指定服务器端口号
  },
  plugins: [
    new HotModuleReplacementPlugin(),
  ]
}

//在对应模块中引入
// 增加，开启热更新之后的代码逻辑
 if (module.hot) {
     //xxx 模块需要热更新
     module.hot.accept(['./xxx'], () => {
       /*
       do something
       */   
     })
 }


```



- **DllPlugin**

https://webpack.wuhaolin.cn/4%E4%BC%98%E5%8C%96/4-2%E4%BD%BF%E7%94%A8DllPlugin.html

webpack 内置已经支持 

DllPlugin: 打包出 dll 文件

DLLReferencePlugin: 使用 dll 文件



##### 环境区分

1.**可用于生产环境的**

优化 babel-loader（明确缓存，要什么不要什么的范围）

IgnorePlugin （最好用，不然打包出来代码比较大）

noParse （提高生产环境打包速度）

happyPack 和 ParallelUglifyPlugin （多进程打包提高效率）

2.**不适用于生产环境**

自动刷新（只是为了提高开发体验）

热更新 （万万不能用于生产环境）

DLLPlugin (生产环境下打包一次就结束了，不需要使用。只是满足开发环境下的一个速度)



#### 2.优化产出代码（重要）

为了线上提高产品性能：让代码体积更小，合理分包不重复加载，速度更快内存使用更小

- **小图片 64 编码**

在 url-loader 的 options 配置。低于多少大小的图片就用 base64 格式产出，不要再进行一次网络请求，很浪费资源

- **bundle加 hash**

在打包出来的 bundle 里面要加上 contentHash。如果说我们的 hash 值没有改变，说明js css 这些文件没有改变，那么上线请求的时候前端代码就会命中缓存，没必要重新加载一遍，可以提高我们的效率

eg：

```js
module.exports = {
	...
	output:{
		filename: '[name].[contentHash:8].js' 
	}
	...
}
```

- **懒加载**

异步 import

- **提取公共代码**
- **IgnorePlugin**
- **使用 CDN 加速**

​	步骤：在 output 中配置 CDN 前缀 => 把打包结果(JS CSS Img 等)上传到 CDN 中

- **使用 production**

1.dev 模式下没有必要压缩代码，只要设置 mode: " production"，就会自动压缩（webpack 4 之后）

2.开发环境下，Vue React 会自动删掉调试代码，比如说开发环境下的 warning

3.自动启动 Tree-Shaking

​	前提：必须使用 ES6 Module 才能让 tree-shaking 生效，不能用 commonjs

​	作用：把没用上的代码（export 出去的）删去

​	为什么只能在ES6 module 条件下：因为 ES6 Module 是静态引入的，编译时就已经引入了，所以有条件分析我们引入	了什么，什么要用什么不用。但是 commonJS 是动态引入的，在编译时我们不知道到底要不要引入这个模块

​	eg

```js

if(dev){
	// require 动态引入 可行
	apiList = require('../config/api/apiList.js')
}

if(dev){
	// import 静态引入，编译时报错
	import apiList from "../config/api/apiList.js"
}

```

- **Scope Hosting**

一个函数有一个作用域，一般来说，webpack 会把一个文件编成一个函数，加上 SH 之后就可以把这些文件编到一个函数中。

配置：

```js
const ModuleConcatenationPlugin = require ('webpack/lib/optimize/ModuleConcatenationPlugin')
module.exports = {
	resolve:{
		mainFields: ['jsnext:main', 'browser', 'main'] //优先使用jsnext:main指向 ES6模块化文件
	}
	plugins: [
		new ModuleConcatenationPlugin();
	]
}
```

#### 3.一些常见问题

##### 前端为什么要进行打包和构建？

两个方面，代码层面和开发层面。对代码进行打包和构建可以：有更小的代码体积，让代码加载更快，比如说 tree-shaking 代码压缩 代码合并等的使用； 可以支持我们使用高级语言和特性，我们在使用的时候不必考虑其它问题，比如说 TS ES6 模块化 scss 这些；可以提高我们代码的兼容性，比如说 polyfill。 在开发层面：打包和构建可以让我们有一个统一的开发环境和标准（比如说都输出成 ES5 或者 ES6），提高整个部门的效率，还可以集成公司构建规范。

##### loader 和 plugin 的区别？

loader 模块转化器，比如把 less 变成 css

plugin 扩展插件，loader 结束之后工作，丰富 webpack 的功能。比如说 loader 只能在最开始的时候工作，但是 plugin 就可以在 webpack 整个编译阶段按需求工作。

##### 常见的 loader 和 plugin

##### babel 和 webpack 的区别

babel 只是个编译工具，只关注语法，不关心模块化，只看语法有没有错误；webpack 是打包构建工具，是多个 loader 和 plugin  的集合。 

##### 如何产生一个 lib

eg

```js
output {
	//lib 的文件名
	filename: "",
	// 输出目录
	path: ,
	// lib 的全局变量名
	library: ""
}
```

##### babel-polyfill 和 babel-runtime 的区别

##### webpack 实现懒加载

import() 语法 + Vue React 异步组件 +Vue-router React-router 异步加载路由

##### 为什么 proxy 不能用 polyfill？



##### webpack有什么常见优化手段？      
