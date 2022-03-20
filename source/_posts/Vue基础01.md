---
title: Vue基础01 
date: 2021-12-20 09:20:05
tags: Vue2 基础知识
---

Vue 的指令、插值等基础的知识汇总

<!-- more -->

##### 1.指令，插值

（1）插值、表达式

（2）指令、动态属性

```js
v-bind:title = 'message'  //绑定元素 attribute 简写 ：
v-if = 'condition'  // 条件语句
v-for = "(index, item) in list" // 循环 注意要有 :key = "" 绑定 key
v-on:click="reverseMessage"  // 绑定事件 简写 @
v-model  // 实现表单输入和应用状态之间的双向绑定

```

```html
// eg  v-model
<div id="two-way-binding">
  <p>{{ message }}</p>
  <input v-model="message" />
</div>

```

```js
const TwoWayBinding = {
  data() {
    return {
      message: 'Hello Vue!'
    }
  }
}
Vue.createApp(TwoWayBinding).mount('#two-way-binding')
```

（3）v-html：绑定的数据可以是富文本，但是有XSS风险，会覆盖子组件

v-html更新的是元素的 innerHTML 。内容按普通 HTML 插入， 不会作为 Vue 模板进行编译 。

XSS风险：https://blog.csdn.net/lj1530562965/article/details/108790220



##### 2.computed 和 watch

（1）`computed` 有缓存，data 不变则不会重新计算

```js
var vm = new Vue({
  data: { a: 1 },
  computed: {
    // 仅读取
    aDouble: function () {
      return this.a * 2
    },
    // 读取和设置
    aPlus: {
      get: function () {
        return this.a + 1
      },
      set: function (v) {
        this.a = v - 1
      }
    }
  }
})
vm.aPlus   // => 2
vm.aPlus = 3   //这里传参了 注意这里是 set 也就是 v = 3
vm.a       // => 2
vm.aDouble // => 4
```

（2）`watch` 如何进行深度监听？

`watch` 里面有 `handler` 函数、 `immediate` 属性、`deep` 属性（默认值是 false，代表是否深度监听）

```js
const app = createApp({
  data() {
    return {
      a: 1,
      b: 2,
      c: {
        d: 4
      },
      e: 5,
      f: 6
    }
  },
  watch: {
    // 侦听顶级 property
    a(val, oldVal) {
      console.log(`new: ${val}, old: ${oldVal}`)
    },
    // 字符串方法名
    b: 'someMethod',
      
    // 该回调会在任何被侦听的对象的 property 改变时被调用，不论其被嵌套多深
    c: {
      handler(val, oldVal) {
        console.log('c changed')
      },
      deep: true
    },
    // 侦听单个嵌套 property
    'c.d': function (val, oldVal) {
      // do something
    },
      
    // 该回调将会在侦听开始之后被立即调用
    e: {
      handler(val, oldVal) {
        console.log('e changed')
      },
      immediate: true
    },
    // 你可以传入回调数组，它们会被逐一调用
    f: [
      'handler1',
      function handle2(val, oldVal) {
        console.log('handler2 triggered')
      },
      {
        handler: function handle3(val, oldVal) {
          console.log('handler3 triggered')
        }
        /* ... */
      }
    ]
  },
  methods: {
    someMethod() {
      console.log('b changed')
    },
    handler1() {
      console.log('handle 1 triggered')
    }
  }
})

const vm = app.mount('#app')

vm.a = 3 // => new: 3, old: 1
```



```js
watch: {
  obj: {
    handler(newVal, oldVal) {
      console.log('obj.a changed');
    },
    immediate: true,
    deep: true
  }
}
```

这样只要 obj 改变就都会深度监听，开销大 

优化，只对 obj.a 进行深度监听

```js
watch: {
  'obj.a': {
    handler(newVal, oldVal) {
      console.log('obj.a changed');
    },
    immediate: true,
    deep: true
  }
}
```

（3）watch 监听引用类型，因为指针是一样的所以拿不到oldVal，

```html
    <div>
        <input v-model="name">
        <input v-model="info.city">
    </div>
```



```js
 export default{
     data(){
         return {
             name: 'zhangsan', // 值类型监听可以直接拿到值
             info: {
                 city: 'Guangzhou'  // 引用类型不可以 
             }
         }
    },
    watch: {
        name(oldVal, val){
            console.log('watch name',oldVal, val);
        },
        info: {
            handler(oldVal, val) {
                console.log('watch info', oldVal, val) // 引用类型，拿不到 oldVal 。因为指针相同，此时已经指向了新的 val,info 改了可以监听到，但是 info里面的就不行了
            },
            deep: true // 深度监听
        }
       
    }
 }
```

##### 3.class 和 style 

（1）使用动态属性 `v-bind` 来绑定 `class` 和 `style`，表达式结果的类型除了字符串之外，还可以是对象或数组。

```html
// 比较常见的用法
    <div>
        <p :class="{black: isBlack,yellow: isYellow}">使用class</p>  
        <p :class="[black,yellow]">使用 class （数组）</p>
        <p :style="styleData">使用 style</p>
    </div>
```

```js

export default {
    data() {
        return {
            isBlack: true,
            isYellow: true,

            black: 'black',
            yellow: 'yellow',

            styleData: {
                fontSize: '40px',
                color: 'red',
                backgroundColor: '#ccc'
            }
        }
    }
}

```

```css
<style scoped>
    .black {
        background-color: #999;
    }
    .yellow{
        color: yellow;
    }
</style>
```

（2）注意多个单词用使用驼峰命名法

##### 4.条件渲染

（1）`v-if` `v-else` 的用法

可以使用变量，也可以使用 === 表达式

（2）`v-if`  和 `v-show` 的区别

`v-if`不渲染DOM； `v-show` DOM树上还是有，只是 `display: none`

（3）`v-if`和 `v-show` 的使用场景

更新频率不高，就用 `v-if`； 切换得比较频繁就用 `v-show` 

```html
    <div>
        <p v-show="type === 'a'">v-show a</p>
        <p v-show="type === 'b'">v-show b</p>
        <p v-if="type  === 'a'">v-if a</p>
        <p v-if="type  === 'b'">v-if b</p>     
    </div>
```



```js
 export default {
     data(){
         return {
             type: 'a'
         }
     }
 }
```

`DOM` 节点情况

![image-20211220192942622](C:\Users\Jiali\AppData\Roaming\Typora\typora-user-images\image-20211220192942622.png)



##### 5.循环（列表）渲染

（1）可以用 `v-for` 来遍历对象

（2）`key` 的重要性: 用 `v-for` 需要用 `key` ，而且不能乱写 `key`

（3）注意 `v-if` 和 `v-for` 不建议一起写

因为 `v-for` 相当于一个循环体，如果结构里面添加了 `v-if` 那么每一次循环都要判断一次，很浪费资源

```html
    <div>
        <p>遍历数组</p>
        <ul>
            <li v-for="(item, index) in listArr" :key="item.id" >
                {{index}}-{{item.id}}-{{item.tittle}}
            </li>
        </ul>

         <p>遍历对象</p>
        //   v-if 不要和 v-for 写在一起
        <ul v-if="flag">
            <li v-for="(val,index,key) in listObj" :key="key">
                {{index}} - {{key}} - {{val.title}}
            </li>
        </ul>
    </div>
```



```js

export default{
    data(){

        return{
            flag: false,
            listArr: [
                // id 作为 key 比较合适
                {id : 'a', title: '标题1'},
                {id : 'b', title: '标题2'},
                {id : 'c', title: '标题3'}
            ],
            listObj: {
                // 本身的 key 就可以作为遍历的 key
                a:{title: '标题1'},
                b:{title: '标题2'},
                c:{title: '标题3'}
            }
        }
    }
}
```

##### 6.事件

（1） `event` 是原生的

有时我们需要访问原始的 `DOM` 事件。可以用特殊变量 `$event` 把它传入方法：

```html
<button v-on:click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>
```

```js
// ...
methods: {
  warn: function (message, event) {
    // 现在我们可以访问原生事件对象
    if (event) {
      event.preventDefault()
       console.log(event)
    }
    alert(message)
  }
}
```

（2）事件被挂载到当前元素上

```html
    <div>
        <p>{{ num }}</p>
        <button @click="increment1"> + 1</button>
        <button @click="increment2(2,$event)"> + 2</button>
    </div>
```



```js

    methods: {
        // 如果只需要 event 参数，绑定的时候可以只写个函数名
        increment1(){
            console.log('event',event,event.__proto__.constructor); // constructor 是原生的 event 对象
            console.log('event',event,event.target); 
            console.log('event',event,event.currentTarget);  // 事件挂载在当前元素上 和 react 是不同的
            this.num++
        },
        // 如果需要别的参数，那么绑定的时候函数里要把参数写全了
        increment2(val,event){
            console.log(event.targrt);
            this.num += val
        }
    }


```

##### 7.事件修饰符

在事件处理程序中调用 `event.preventDefault()` 或 `event.stopPropagation()` 是非常常见的需求。尽管我们可以在方法中轻松实现这点，但更好的方式是：方法只有纯粹的数据逻辑，而不是去处理 `DOM` 事件细节。

为了解决这个问题，`Vue.js` 为 `v-on` 提供了**事件修饰符**。 

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`
- `.passive`

```vue
		<!-- 阻止单击事件继续传播 -->
        <a v-on:click.stop="doSomething"></a>
        <!-- 提交事件不再重载页面 -->
        <form v-on:submit.prevent="onSubmit"></form>
        <!-- 修饰符可以串联 -->
        <a v-on:click.stop.prevent="doSomething"></a>
        <!-- 只有修饰符 -->
        <form v-on:submit.prevent></form>
        <!-- 添加事件监听器使用事件捕获模式 -->
        <!-- 即内部元素触发的事件先在此处理，然后再交由内部元素进行处理 -->
        <div v-on:click.capture="doSomething">...</div>
        <!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
        <!-- 即事件不是从内部元素触发的 -->
        <div v-on:click.self="doSomething">...</div>
```

##### 8.按键修饰符

```vue
		8<!-- 即使 Alt 和 Shift 被一同按下时也会被触发 -->
        <button @click.ctrl="onClick">A</button>
        <!-- 有且只有当 Ctrl 被按下时才会被触发 -->
        <button @click.ctrl.exact="onCtrlClick">A</button>
        <!-- 没有任何系统修饰符被按下时才会被触发 -->
        <button @click.exact="onClick">A</button>
```

##### 9.表格（用 v-model 绑定数据）

##### 9.1 输入元素

`v-model` 在内部为不同的输入元素使用不同的 property 并抛出不同的事件：

- text 和 textarea 元素使用 `value` property 和 `input` 事件；
- checkbox 和 radio 使用 `checked` property 和 `change` 事件；
- select 字段将 `value` 作为 prop 并将 `change` 作为事件。

##### 9.2 对于复选框

单个复选框，绑定到布尔值

多个复选框，绑定到同一个数组

```vue
<template>
    <div>
        <p>输入框: {{name}}</p>
        <input type="text" v-model.trim="name"/>  // 截取
        <input type="text" v-model.lazy="name"/>  // 防抖
        <input type="text" v-model.number="age"/> // 转换为数字

        <p>多行文本: {{desc}}</p>
        <textarea v-model="desc"></textarea>  // 用 v-model 来实现输入框
        <!-- 注意，<textarea>{{desc}}</textarea> 是不允许的！！！ -->

        <p>复选框 {{checked}}</p>
        <input type="checkbox" v-model="checked"/>  

        <p>多个复选框 {{checkedNames}}</p>
        <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
        <label for="jack">Jack</label>
        <input type="checkbox" id="john" value="John" v-model="checkedNames">
        <label for="john">John</label>
        <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
        <label for="mike">Mike</label>

        <p>单选 {{gender}}</p>  
        <input type="radio" id="male" value="male" v-model="gender"/>
        <label for="male">男</label>
        <input type="radio" id="female" value="female" v-model="gender"/>
        <label for="female">女</label>

        <p>下拉列表选择 {{selected}}</p>
        <select v-model="selected">
            <option disabled value="">请选择</option>
            <option>A</option>
            <option>B</option>
            <option>C</option>
        </select>

        <p>下拉列表选择（多选） {{selectedList}}</p>
        <select v-model="selectedList" multiple>
            <option disabled value="">请选择</option>
            <option>A</option>
            <option>B</option>
            <option>C</option>
        </select>
    </div>
</template>

<script>
export default {
    data() {
        return {
            name: 'zhangsan',
            age: 18,
            desc: '自我介绍',
            checked: true,
            checkedNames: [],
            gender: 'male',
            selected: '',
            selectedList: []
        }
    }
}
</script>
```

