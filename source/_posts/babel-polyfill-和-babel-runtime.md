---
title: babel-polyfill 和 babel-runtime
date: 2022-02-27 18:32:11
tags: babel babel-polyfill babel-runtime
---

babel基本配置和  babel-polyfill babel-runtime 简介

<!--more -->

##### 1.babel基本配置

安装

```js
"devDependencies": {
    "@babel/cli" ,
    "@babel/core" ,
    "@babel/plugin-transform-runtime" ,
    "@babel/preset-env"
  },
  "dependencies": {
    "@babel/polyfill",
    "@babel/runtime"
  }
```

预设 和 plugins （按需）

```js
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "useBuiltIns": "usage",
                "corejs": 3
            }
        ]
    ],
    "plugins": [
        [
            "@babel/plugin-transform-runtime",
            {
                "absoluteRuntime": false,
                "corejs": 3,
                "helpers": true,
                "regenerator": true,
                "useESModules": false
            }
        ]
    ]
}
```

##### 2.babel-polyfill

- 为了解决浏览器的兼容问题，

  在 babel 7.4 以后不用 babel-polyfill，而是采用以下两个polyfill 集合：core-js 和 regenerator(支持 generator 函数)

- 按需引用 babel-polyfill

```js
"presets": [
        [
            "@babel/preset-env",
            {
			// 加入下面这两个之后就行了，不用再在文件中 import '@babel/polyfill'.
             // 编译之后就会自动 require需要的模块
                "useBuiltIns": "usage",
                "corejs": 3
            }
        ]
    ],
```

- 全局污染

比如说 promise，babel-polyfill 的方式是重新再 window 上定义一个 promise

```js
window.promise = function(){...}
```

如果说我们在使用的时候（类似开发第三方库这种），自己重写了 window.pormise，那么就会导致冲突

##### 3.babel-runtime

babel-polyfill 的改进，配置 plugins 下

```js
[
            "@babel/plugin-transform-runtime",
            {
                "absoluteRuntime": false,
                "corejs": 3,
                "helpers": true,
                "regenerator": true,
                "useESModules": false
            }
        ]
```

比如说 promise， babel-runtime 会重新定义为 _promise = xxx，不会污染全局环境 
