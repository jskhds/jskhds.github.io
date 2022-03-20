---
title: Vue原理01
date: 2021-12-23 15:14:03
tags: 组件化 响应式 vdom diff 模板编译 渲染过程 前端路由
---

Vue原理从大方向上有三点：响应式、模板编译、`Vdom`，这一节主要内容为 响应式 和 `Vdom`

<!-- more -->

#### 1.组件化基础（MVVM模型）

- 数据驱动视图（`MVVM` 和 `setState`）

组件化在很久以前就已经有了，比如说 web 1.0 php 之类的，但是当时需要频繁操作 DOM，所以 jquery 比较流行（因为操作 DOM 很多）。传统组件只是静态渲染，更新还要依赖于操作 DOM。但是在 Vue 或者 React 中，只要修改数据就可以了，也就是更多的关注业务逻辑，可以实现很复杂的页面功能

- `Vue MVVM` 模型图（画出来）`Model-View-ViewModel`

  `DOM` →`vue` （监听/指令） → 纯 `javascript` 对象

<img src="../images/mvvm.png" style="zoom: 33%;" />

- 示例

```html
	<div>
        <p @click="changeName">{{name}}</p>
    </div>
```

```js
data(){
        return {
             name: "zhangsan"
    	},
    		methods: {
        	changeName(){
                this.name = "lisi"
            }
    }
```

`view`: `html` 里的 `DOM` 

`Model`: `data`

`ViewModel`: `@click` 和 `methods` （`ViewModel` 不好单独说是什么，但是总体上它就是 `View` 和 `Model` 之间的桥梁，`model` 修改了之后就可以修改 `view` ）



#### 2.Vue 响应式（很重要）

响应性是一种允许我们以声明式的方式去适应变化的编程范例。在 `vue`中，就是 `data` 的数据一旦变化，视图立马更新，这也是实现数据驱动视图的第一步 

##### 2.1 核心 API - Object.defineProperty

`JS` 中的 `Object.defineProperty`  是 `Vue` 实现响应式的核心 `API`

```js
// Object.defineProperty(obj, prop, descriptor)  对象 obj 拥有了 prop 属性
const data = {}
let nickName = "Dmoon"
Object.defineProperty(data,'nickName',{
    get:function(){
        console.log('get');
        return nickName;
    },
    set:function(newVal){
        console.log('set');
        nickName = newVal
    }

})
console.log(data.nickName); // 调用 get
data.nickName = 'moon'  // 调用 set
```

深度监听

两个主要函数：`defineReactive` 和 `observer`

```js
// 触发更新视图
function updateView(){
    console.log("视图更新");
}
function defineReactive(target, key, value){
    // 深度监听,递归调用到 value 的最后一层才返回
    observer(value)
    // 核心 API 监听数据变化
    // get(返回数据) 和 set(修改数据) 
    Object.defineProperty(target, key ,{
        get(){
            return value
        },
        set(newValue){
            if(newValue !== value){
                // 设置新值的时候，也需要监听。
                // data.age = {num: 21},如果不设置那么就调用不到下一层， num 也不能作为 key 传进来
                observer(value)  
                // 设置新值
                // value 一直在闭包中，所以get可以拿到新值
                value = newValue
                updateView()
            }
        }
    })

}

// 监听对象属性
function observer(target){
    // 只监听 对象或数组
    if(typeof target !== 'object' || target == null){
        return target
    }
    // 重新定义各个属性 for in 可以遍历数组也可以遍历对象
    for(let key in target){
        defineReactive(target, key, target[key])
    }
}

// 准备数据
const data = {
    name: 'zhangsan',
    age: 20,
    info:{
        address:{
            city: 'beijing'  // 需要深度监听
        }
    }
}

// 监听数据
observer(data)

// 测试
data.name = 'lisi'
data.age = 18
data.info.address.city = 'shanghai'
console.log('age',data.age);
console.log(data.info.address.city);



```

##### 2.2 Object.defineProperty 的一些缺点（3.0 启用 Proxy）

- 不管原数组/对象有多深，都需要一次性递归到底，计算量很大
- 无法监听新增属性/删除属性（`Vue.set Vue.delete`）
- 不能原生监听数组，需要特殊处理

如果要监听数组，需要避免污染数组原型，所以先另外声明一个数组原型，再 `Object.create()`指向新声明的数组原型。比如说我们在 `push`时需要触发更新，那么像这样处理后，我们可以在新对象里重写 push 函数并且还可以调用本来数组原型里的 push 函数。

```js
// 监听数组时，我们需要在外面单独声明一个原型，以免污染全局的原型链
const oldArrayProperty = Array.prototype
// Object.create 创建新对象，原型指向 oldArrayProperty，再扩展新的方法，不会影响原型
// 比如说我们定义 function push(){console.log('push')} 和 arrProto.__proto__.push(4) 
// 前者调用自定义函数，后者是原型链上的属性，二者不矛盾
const arrProto = Object.create(oldArrayProperty);
// 重写常用的方法
['push', 'splice', 'pop', 'shift', 'unshift'].forEach(methodeName =>{
    arrProto[methodeName] = function(){
        updateView()
        oldArrayProperty[methodeName].call(this,...arguments)
        // 相当于 Array.prototype.push.call(this, ...arguments)
    }
})
```

同时要修改 `observer` 函数，其实就添加一句判断是不是 Array 让其指向我们定义的新数组原型

```js
function observer(target) {
    if (typeof target !== 'object' || target === null) {
        return target
    }
    // 不可以把原型定义在函数里面，不然会污染全局的 Array 原型
    // Array.prototype.push = function () {
    //     updateView()
    //     ...
    // }
	// 判断是不是数组 如果是 添加 __proto__
    if (Array.isArray(target)) {
        target.__proto__ = arrProto
    }
    for (let key in target) {
        defineReactive(target, key, target[key])
    }
}

```

#### 3.虚拟DOM（virtual dom）和 diff（很重要）

`vdom` 是实现 `vue` 和 `react` 的重要基石，`diff` 算法是 `vdom` 中最核心最关键的部分。我们都知道 `DOM` 操作很耗时，之前我们用 `jquery` 手动调整 `DOM` 操作，而 `Vue` 和 `React` 是数据驱动视图，技术复杂度下来了但是业务复杂度上去了，所以如何有效控制 `DOM` 操作仍然是一个重要课题。

##### 3.1 解决方案-vdom

复杂度上来了，想减少计算次数是不大可能的，但是我们可以把计算转移给 JS 来算，因为 JS 执行速度很快。进一步的，我们可以用 JS 来模拟 DOM 结构，**计算出最小的变更来操作 DOM**，这就叫 `VDom`。其中 **diff 算法**是最核心最关键的部分。

- 示例：用 VNode 来模拟下面这个片段

```html
<div id="div1" class="container">
        <p>vdom</p>
        <ul style="font-size: 20px;">
            <li>a</li>
        </ul>
    </div>
```

虚拟dom（示例，不一定都是这样写的，**但是要写得出来这个结构**）

有 tag props children 

```js
{
            tag: 'div',
            props: {
                className: 'container',
                id: 'div1'
            },
            children: [
                {
                    tag: 'p',
                    children: 'vdom'
                },
                {
                    tag: 'ul',
                    props: {style: 'font-size: 20px'},
                    children: [
                        {
                            tag: 'li',
                            children: 'a'
                        }
                       // ...
                    ]
                }
            ]

        }
```

- 通过 `snabbdom` 来学习 `vdom`

 重点：h 函数（产生 vnode）、vnode 数据结构、patch 函数（两个参数都是node 或者 渲染和 node）

patch函数的重点： **patch(elem, vnode)**  and **patch(vnode, newVnode)**

```js
import {
  init,
  classModule,
  propsModule,
  styleModule,
  eventListenersModule,
  h,
} from "snabbdom";

const patch = init([
  // Init patch function with chosen modules
  classModule, // makes it easy to toggle classes
  propsModule, // for setting properties on DOM elements
  styleModule, // handles styling on elements with support for animations
  eventListenersModule, // attaches event listeners
]);


const container = document.getElementById("container");
// vnode 
const vnode = h("div#container.two.classes", { on: { click: someFn } }, [
  h("span", { style: { fontWeight: "bold" } }, "This is bold"),
  " and this is just normal text",
  h("a", { props: { href: "/foo" } }, "I'll take you places!"),
]);
// Patch into empty DOM element – this modifies the DOM as a side effect
patch(container, vnode);

const newVnode = h(
  "div#container.two.classes",
  { on: { click: anotherEventHandler } },
  [
    h(
      "span",
      { style: { fontWeight: "normal", fontStyle: "italic" } },
      "This is now italic type"
    ),
    " and this is still just normal text",
    h("a", { props: { href: "/bar" } }, "I'll take you places!"),
  ]
);
// 把 vnode 渲染到 container 上
patch(container, vnode);

// 更新数据
const newVnode = h(
  "div#container.two.classes",
  { on: { click: anotherEventHandler } },
  [
    h(
      "span",
      { style: { fontWeight: "normal", fontStyle: "italic" } },
      "This is now italic type"
    ),
    " and this is still just normal text",
    h("a", { props: { href: "/bar" } }, "I'll take you places!"),
  ]
);
// 更新已有的内容
patch(vnode, newVnode); // Snabbdom efficiently updates the old view to the new state
```



##### 3.2 diff算法

**diff算法 ** 帮助`vnode` 计算出 `DOM` 最小的改变

diff 算法之前复杂度为 O(n^3) ，后来经过优化达到了 O(n) 次方，主要是三点：

- 只比较同一层级，不跨级比较；


- tag 不相同之间删掉重建，不再深度比较；


- tag 和 key 二者相同时，则认为是相同节点，不再比较

##### 3.3 snabbdom 源码解读

h函数对数据进行处理后，传递给 vnode(sel, data, children, text)，返回 vnode；  

vnode 函数返回一个对象， {sel, data, children, text, elm, key };

patch 函数 patch(container, vnode) patch(vnode,newVnode)

- `patchVnode`
- `addVnodes` `removeVnos`
- `updateChildren`函数( key 的重要性)

https://juejin.cn/post/6994959998283907102#heading-8

**https://juejin.cn/post/6984049109267120136**















