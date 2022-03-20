---
title: webpack基本使用与配置
date: 2022-02-26 15:29:39
tags: webpack 前端打包
---

webpack 的基本使用与配置

<!--more-->

#### 基本配置

##### **1.拆分配置 和 merge**

一般来说配置文件有：

```js
webpack.common.js  //公共配置
webpack.dev.js     // 开发配置
webpack.prod.js   //线上配置
paths               // 配置路径
```

##### **2.启动本地服务(dev.js)**

插件：webpack-dev-server

设置代理：proxy

module.exports/devServer

```js
// 配置前端的代理，比如说当前前端项目的端口是 8080 所以我们在请求接口的时候，
    // 可以设置成 http://localhost:8080/api/users 这种类型.简写成 /api/users 也可以
    // 因为代理设置的标价是 /api 所以前端遇到包含api的请求以后， 就定向到 http://localhost:3000 来
    // 也就是最终到 http://localhost:3000/api/users 去请求资源
    // 如果后端的地址不是这样的，可能是 http://localhost:3000/users 或者 http://localhost:3000/info/users这种和前端有出入的地址
    // 那么可以添加 pathRewrite 属性，pathRwrite:{"^/api": "替换的值，空或者其它对应字符串"}
    proxy:{
      "/api":{
        target:"http://localhost:3000"
      }
    }
  },
```

##### **3.处理ES6(common.js)**

插件 babel-loader

```js
rules:  {
                test: /\.js$/,
                loader: ['babel-loader'],
                include: srcPath,
                exclude: /node_modules/
            },
```

预设：.babelrc

```js
{
    "presets": ["@babel/preset-env"],
    "plugins": []
}
```

##### **4.处理样式**

PS注意处理顺序

```js
		 rules:{
                test: /\.css$/,
                // loader 的执行顺序是：从后往前 postcss-loader做浏览器兼容
                loader: ['style-loader', 'css-loader', 'postcss-loader'] // 加了 postcss
            },
            {
                test: /\.less$/,
                // 增加 'less-loader' ，注意顺序
                loader: ['style-loader', 'css-loader', 'less-loader']
            }
```

对于 postcss-loader 需要 autoprefixer，处理css 的一些新特性，比如说 transform 之类的

根目录下新建postcss.config.js

```js
module.exports = {
    plugins: [require('autoprefixer')]
}
```

##### **5.处理图片**

开发

```js
rules: [
            // 直接引入图片 url
            {
                test: /\.(png|jpg|jpeg|gif)$/,
                use: 'file-loader'
            }
        ]
```

上线

```js
rules: [
            // 图片 - 考虑 base64 编码的情况
            {
                test: /\.(png|jpg|jpeg|gif)$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        // 小于 5kb 的图片用 base64 格式产出
                        // 否则，依然延用 file-loader 的形式，产出 url 格式
                        limit: 5 * 1024,
                        // 打包到 img 目录下
                        outputPath: '/img1/',

                        // 设置图片的 cdn 地址（也可以统一在外面的 output 中设置，那将作用于所有静态资源）
                        // publicPath: 'http://cdn.abc.com'
                    }
                }
            },
        ]
```

##### **6.模块化**

webpack本身就是模块化的，但是像 import 语法等浏览器是不支持识别的，所以我们在使用的时候还需要引入 webpack 本身。

##### **7.一些插件**

`new CleanWebpackPlugin()`:再次打包时把之前打包的文件自动删除，然后打包新的代码；

`new HtmlWebpackPlugin()`: 在打包结束后，自动生成一个 html 文件，并把打包生成的 js 模块引入到该 html 中

#### 高级配置

##### **1.多入口**

步骤：entry => output => plugins

```js
	//input 在entry中配置
entry: {
        index: path.join(srcPath, 'index.js'),
        other: path.join(srcPath, 'other.js')
 },

	//ouput
output: {
        // filename: 'bundle.[contentHash:8].js',  // 打包代码时，加上 hash 戳
        filename: '[name].[contentHash:8].js', // name 即多入口时 entry 的 key
        path: distPath,
        // publicPath: 'http://cdn.abc.com'  // 修改所有静态文件 url 的前缀（如 cdn 域名），这里暂时用不到
 },
	//plugins 每个入口文件都要配置自己的 new HtmlWebpackPlugin 生成 html
     // 多入口 - 生成 index.html
        new HtmlWebpackPlugin({
            template: path.join(srcPath, 'index.html'),
            filename: 'index.html',
            // chunks 表示该页面要引用哪些 chunk （即上面的 index 和 other），默认全部引用
            chunks: ['index']  // 只引用 index.js,记得要写这一项，确保只引用特定的文件
        }),
        // 多入口 - 生成 other.html
        new HtmlWebpackPlugin({
            template: path.join(srcPath, 'other.html'),
            filename: 'other.html',
            chunks: ['other']  // 只引用 other.js
        })
```

##### **2.抽离CSS文件**

步骤：区分不同环境配置 => 替代 style-loader => 压缩 css

dev环境下配置不变，prod 配置环境下，需要用 `MiniCssExtractPlugin.loader` 替代 `style-loader`;

```js
{
      test: /\.css$/,
      loader: [
      MiniCssExtractPlugin.loader,  // 注意，这里不再用 style-loader
      'css-loader',
      'postcss-loader'
      ]
   },
 {
     test: /\.less$/,
     loader: [
         MiniCssExtractPlugin.loader,  // 注意，这里不再用 style-loader
         'css-loader',
         'less-loader',
         'postcss-loader'
     ]
 }
```

对于插件 `MiniCssExtractPlugin`的配置

```js
 new MiniCssExtractPlugin({
            filename: 'css/main.[contentHash:8].css'  // 输出的路径，[name].[hash].css
})
```

压缩 css

```js
optimization: {
        minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})],
}
```

##### **3.抽离公共代码**

一些公共引用文件，第三方模块等，希望在更改代码的时候可以让它们命中缓存加载而不是重新再来一遍。

common 和 dev 都没什么改变，还是在 prod 下面更改

```js
splitChunks: {
        cunks: 'all',
       /*
        initial 入口 chunk，对于异步导入的文件不处理
         async 异步 chunk，只对异步导入的文件处理
         all 全部 chunk
       */

        缓存分组
       cheGroups: {
         // 第三方模块
         vendor: {
             name: 'vendor', // chunk 名称
             priority: 1, // 权限更高，优先抽离，重要！！！
             test: /node_modules/,
             minSize: 0,  // 大小限制，按照自己需要的写
             minChunks: 1  // 最少复用过几次，比如说 lodash 这种第三方模块，只要引用过一次就要拆分出来
         },

         // 公共的模块
         common: {
           name: 'common', // chunk 名称
                  priority: 0, // 优先级
                  minSize: 0,  // 公共模块的大小限制
                  minChunks: 2  // 公共模块最少复用过几次
              }
       }
        }
```

代码分割以后，我们就可以给不同的文件在 ` new HtmlWebpackPlugin` 时的 chunks 项按需引用分割后的模块。格式：

`chunks;[ ..., 'common', 'vendor'  ] `



##### **4.实现异步懒加载**

在 import 的时候异步引入，不需要 webpack 额外配置，内在支持

```js
// 加载动态数据
setTimeout(()=>{
    import('./dynamic-data.js').then(res=>{
        console.log(res.default.message) // 因为动态数据导出时是 default 所以这里也要加 default
    })
})
```



##### **5.处理 vue 和 jsx**

jsx 用 babel 就可以，在 .babelrc 中添加支持 react 的预设

```js
{
	"preset":["@babel/preset-react"]
}
```

vue 用 vue-loader 即可

#### **module chunk bundle 的区别**（重要）

module => 各个源码文件，webpack 中一切皆模块

chunk => 多模块合成的，如 entry import() splitChunk

bundle => 最终的输出文件



#### 构建流程概述

- 初始化：从配置文件和 `shell`语句中读取与合并参数，并初始化需要使用的插件和配置插件等执行环境需要的参数、
- 编译构建：从 Entry 出发，针对每个 Module 串行调用的对应的 Loader 去翻译文件内容，再找到该 Module 依赖的 Module，递归地进行编译处理
- 输出流程：对编译后的 Module 组合成 Chunk，把 Chunk 转换成文件，输出到文件系统

#### 

##### 
