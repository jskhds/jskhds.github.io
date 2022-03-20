---
title: Vue原理02
date: 2021-12-25 15:34:33
tags: 模板编译 Vue的组件渲染和更新过程
---

Vue 原理的第三点：模板编译

<!-- more -->

#### 0.过程

`vue template compiler 将模板编译成 render 函数, 执行 render 函数生成 vnode`

####  1. 前置知识

JS 的 with 语法，不常用，不需要深究

```js
const obj = {a: 100, b: 200}
console.log(obj.a)
console.log(obj.b)
console.log(obj.c) // undefined

// with 语法 改变 {} 内变量的查找方式，将 {} 内的自由变量，当做 obj 的属性来查找
// 要慎用 with ，因为它打破了 作用域语法，易读性变差
with(obj){
	console.log(a)  // 默认找 obj.a 而不是 a
	console.log(b)  // 默认找 obj.b
	console.log(c)  // 报错
}
```

#### 2. 模板编译

我们都知道，html 是标签语言，不像 JS 一样，可以实现判断、循环。而我们的模板里面，既有标签，也有指令和插值，还有 JS 表达式，所以模板肯定是转换为某种 JS 代码，即编译模板。

这个转换的过程主要可以分为三步

- 模板编译为 `render` 函数，执行 `render` 函数返回 `vnode`
- 基于 `vnode` 再执行 `patch` 和 `diff`
- 使用 `webpack vue-loader` 在开发环境下编译模板

比如说

##### 2.1.简单的表达式

```js
const compiler = require('vue-template-compiler')
```

```jsx
 const template = `<p>{{message}}</p>`
 const res = compiler.compile(template)
 console.log(res.render);

// with(this){return createElement('p',[createTextVNode(toString(message))])}
// with 里面的 this 就是 const vm = new Vue({...}) 
```

##### 2.2事件

```jsx
// 事件
 const template = `<button @click="submitHandler">submit</button>`
// with(this){return _c('button',{on:{"click":submitHandler}},[_v("submit")])}
/
```

##### 3.3v-model

```js
// v-model 
 const template = `<input type="text" v-model="name">`
// with(this){return _c('input',{directives:[{name:"model",rawName:"v-model",value:(name),expression:"name"}],attrs:{"type":"text"},domProps:{"value":(name)},on:{"input":function($event){if($event.target.composing)return;name=$event.target.value}}})}       

```

看 on 后面的语句， input 已经绑定了 on 事件，触发的时候返回 name=$event.target.value，因为已经使用了 with 语法，所以 name 查找的是 this.name，只要变量更新就会触发新的渲染

补充

```js
function installRenderHelpers (target) {
     target._o = markOnce;
     target._n = toNumber;
     target._s = toString;
     target._l = renderList;
     target._t = renderSlot;
     target._q = looseEqual;
     target._i = looseIndexOf;
     target._m = renderStatic;
     target._f = resolveFilter;
     target._k = checkKeyCodes;
     target._b = bindObjectProps;
     target._v = createTextVNode;
     target._e = createEmptyVNode;
     target._u = resolveScopedSlots;
     target._g = bindObjectListeners;
     target._d = bindDynamicKeys;
     target._p = prependModifier;
}
```



#### 3.Vue 组件中使用 render 代替 template 

可以用 render 函数来写模板

```js
// 假设 this.level = 1 
// 意思就是 h1 标签 下面有一个子元素 a 标签——name 是 headerId， 链接是 '#' + 'headerId'， text 是 'this is a tag'
Vue.component('heading',{
    // template: `xxxx `
    render: function(createElement){
        return createElement(
            'h' + this.level,
            [
                createElement('a', {
                    attrs: {
                        name: 'headerId',
                        href: '#' + 'headerId'
                    }
                }, 'this is a tag')
            ]
        )
    }
})

```

