---
title: Vue原理03
date: 2021-12-26 13:43:13
tags: 渲染更新过程 前端路由
---

Vue 的渲染更新过程 、前端路由原理

<!-- more -->

#### 1.Vue 的渲染更新过程

##### 1.1初次渲染过程

- 解析模板为 render 函数（或在开发环境中已完成，vue-loader）
- 触发响应式(Object.definProperty)，监听 data 属性 getter 和 setter
- 执行 render 函数，生成 vnode, patch(elem,vnode)

这一步要看数据和 dom 是怎么绑定的

```html
<p>{{message}}</p>
```

因为只有 message 绑定上来了，所以在初次渲染的时候，只会触发 message 的get ，像 name age 这些不管怎么变都不会影响到初次渲染的流程 

```js

export default{
	data(){
		return{
			info: {
				message: '人物信息',
				name: 'zhangsan',
				age: '18'
}}}}

```

##### 1.2更新函数

- 修改 data 触发 setter（此前在 getter 中已被监听）
- 重新执行 vnode 函数，生成 newVnode
- patch(vnode, newVnode)

##### 1.3示意图

<img src="..\images\vue-data.png" alt="vue-data" style="zoom:50%;" />

##### 1.4异步渲染（也很重要）

- nextTick

```js
export default{
     data(){
         return {
              list: ['a', 'b', 'c', 'd']
         }
     },
     methods: {
         addItem(){
// 由于 vue 是异步渲染的，所以这里的 data 不管改变多少次，vue 都只会在 data 全部修改完之后进行一次渲染 
             this.list.push(`${Date.now()}`)
             this.list.push(`${Date.now()}`)
             this.list.push(`${Date.now()}`)
             
            const ulElem = this.$refs.ul1
            // 没有nextTick时 我们只能获得修改前的数据
            console.log('没有nextTick时 '+ ulElem.childNodes.length);

            this.$nextTick(()=>{
                 const ulElem = this.$refs.ul1
            console.log('有nextTick时 '+ ulElem.childNodes.length);
            })
         }
     }
 }
```

- 汇总 data 的修改，一次性更新视图
- 减少 DOM 操作次数，提高性能

##### 2.前端路由原理

网页url组成部分

```js
// http://127.0.0.1:8000/hash.html?a=100&b=20#/aaa/bbb
location.protocol //http
location.hostname //127.0.0.1
location.host //127.0.0.1:8000
location.port // 8000
location.pathname //hash.html
location.search // ?a=100&b=20
location.hash // #/aaa/bbb
```

###### 1.hash 的特点

- hash 变化会触发网页跳转，即浏览器的前进后退
- hash 变化不会刷新页面，SPA 必需的特点
- hash 永远不会提交到 server 端

代码演示

```html
<body>
    <p>hash test</p>
    <button id="btn1">修改 hash</button>

    <script>
        // hash 变化，包括：
        // a. JS 修改 url
        // b. 手动修改 url 的 hash
        // c. 浏览器前进、后退
        window.onhashchange = (event) => {
            console.log('old url', event.oldURL)
            console.log('new url', event.newURL)

            console.log('hash:', location.hash)
        }

        // 页面初次加载，获取 hash
        document.addEventListener('DOMContentLoaded', () => {
            console.log('hash:', location.hash)
        })

        // JS 修改 url
        document.getElementById('btn1').addEventListener('click', () => {
            location.href = '#/user'
        })
    </script>
</body>
```

###### 2.H5 history 的特点

- 用 url 规范的路由，但跳转时不刷新页面
- `history.pushState`
- `window.onpopstate`

代码演示

```js
<body>
    <p>history API test</p>
    <button id="btn1">修改 url</button>

    <script>
        // 页面初次加载，获取 path
        document.addEventListener('DOMContentLoaded', () => {
            console.log('load', location.pathname)
        })

        // 打开一个新的路由
        // [注意]用 pushState 方式，浏览器不会刷新页面
        document.getElementById('btn1').addEventListener('click', () => {
            const state = { name: 'page1' }
            console.log('切换路由到', 'page1')
            history.pushState(state, '', 'page1') // 重要！！
        })

        // 监听浏览器前进、后退
        window.onpopstate = (event) => { // 重要！！
            console.log('onpopstate', event.state, location.pathname)
        }

        // 需要 server 端配合，可参考
        // https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90
    </script>
</body>
```

###### 3.选择

to B 推荐 hash，简单易用对url不敏感

to C 可以用 H5 history

选简单的

