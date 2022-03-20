---
title: Vue基础03
date: 2021-12-22 14:17:56
tags: vue2 高级特性
---

`Vue` 的一些高级特性 自定义 `v-model` 、插槽、异步组件等

<!-- more -->

#### 1.自定义 v-model

`v-model` 数据双向绑定，输入 `input` 的数据变化时，`v-model` 绑定的数据也会变化

定义一个 `CustomVModel` 组件 ，双向绑定的数据是 `name` 

```html
	<div>
        <p>{{name}}</p>       
        <CustomVModel v-model="name"/>
        <button @click="getName()">名字</button>
    </div>
```

```js
import CustomVModel from './CustomVModel'
export default{
    components: {
        CustomVModel
    },
    data(){
        return{
            name: 'zhangsan'
        }
    },
    methods:{
        getName(){
            console.log(this.name)
        }
    }
}
```

自定义的组件 `CustomVModel` :

实际上是一个 `input`， 拆分成了 `value`属性 和 `input` 事件

```html
        <input type="text" 
            :value="text"
            @input="$emit('change', $event.target.value)"
        >
        <!-- 
            1.上面的 input 使用了 :value 而不是 v-model
            2. change 要和 model.event 里面的 change 对应
            3. text 对应 model.prop 里面的 text
         --> 
```



```js
 export default{
     model: {
         prop: 'text' ,// 和 input 中的 :value 以及 props 中的 text 对应
         event: 'change'  // 和 input 的 change 对应
     },
     props: {
         text: String,
         default(){
             return ''
         }
     },
     data(){
         return {

         }
     }
 }
```

#### 2.$nextTick

`Vue` 是异步而不是同步渲染，所以当 `data` 改变后，`DOM` 不会立即渲染

`$nextTick` 会在 `DOM` 渲染之后触发，以获取最新 `DOM` 节点

以下这段代码就是说：向 `list` 里 `push` 元素后，我们立马拿到元素长度，只能拿到渲染以前的不能拿到最新的；而 `$nextTick` 会在 `DOM` 渲染后才开始执行，所以在 `$nextTick` 里面执行才能获得最新的长度

```vue
<template>
    <div>
        <!-- ref 属性获得 DOM 节点 -->
        <ul ref="ul1">
            <li v-for="(item, index) in list" :key="index">
                {{item}}
            </li>
        </ul>
        <button @click="addItem">添加</button>
    </div>
</template>

<script>
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
            console.log('没有nextTick时 '+ ulElem.childNodes.length);

            this.$nextTick(()=>{
                 const ulElem = this.$refs.ul1
            	console.log('有nextTick时 '+ ulElem.childNodes.length);
            })
         }
     }
 }
</script>

```

#### 3.slot (这部分里面最常用的)

##### 3.1基本使用

 父组件往子组件中插入一段内容。比如说下面这个例子，在父组件里使用子组件的时候，我们希望可以让子组件显示一段内容(直接使用父组件的数据渲染，没有传值的操作)。那么子组件就可以在内部写 `slot` 插槽来放置这段内容。 

`index`

```vue
<template> 
<div>
        <SlotDemo :url="website.url">
             {{website.title}}
        </SlotDemo>
    </div>
</template>
<script>
import SlotDemo from './SlotDemo'
export default{
    components: {
        SlotDemo
    },
    data(){
        return{
             name:'zhangsan',
             website:{
                 title:'百度',
                 url:'https://www.baidu.com/',
                 subTitle: '全球中文第一搜索引擎'
             }
           
        }
    }
}
</script>
```

`SlotDemo`

```vue
<template>
<div>
        <a :href="url">
            <slot>
                接收父组件的内容，如果父组件没有传的话，就默认显示这一行
            </slot>
        </a>
    </div>
</template>

<script>
export default{
    props:   ['url'],
    data(){
        return {}
    }
}
   </script>
```



##### 3.2作用域插槽 

父组件可以通过插槽拿到子组件的数据。子组件在 <slot></slot> 标签中把自己的数据传递出去，即<slot :slotData="要传递的数据"></slot>

父组件在使用子组件时插入一个 <template></template> 标签，使用`v-slot`接受子组件传递过来的数据，即 <template v-slot="slotProps">{{`要渲染的数据`}}</template> 

注意两点，1.`slotData slotProps` 都是自定义的名字；2.slotProps 和 slotData 传递过来的数据不是一个东西，父组件接收到的是一个对象，其中有子组件传过来数据的这个属性。详细可以看代码

```vue
// index
<template>
    <div>
        <ScopedSlot :url = "website.url">
            <!-- 注意里面要写 template 标签 v-slot 绑定的名字也随意 -->
            <template v-slot="slotProps">  
                <!-- 这样父组件可以拿到子组件的 title  -->
                {{slotProps.slotData.title}}
                <!--  
				slotProps:{ "slotData": 
							{ "title": "谷歌", 
							  "url": "https://www.google.com/", 															  "subTitle": "全球英文第一搜索引擎" 
							} 
						  }
				-->
            </template>
        </ScopedSlot>
    </div>
</template>

<script>
import ScopedSlot from './ScopedSlot'
export default{
    components: {
        ScopedSlot
    },
    data(){
        return{
             name:'zhangsan',
             website:{
                 title:'百度',
                 url:'https://www.baidu.com/',
                 subTitle: '全球中文第一搜索引擎'
             }
           
        }
    }
}
 
</script>

```

`ScopedSlot`

```vue
<template>
     <a :href="url">
         <!-- 定义一个动态属性（属性名随意） 绑定组件的数据 -->
        <slot :slotData="website">
            
        </slot>
    </a>
</template>

<script>
 export default{
     props: ['url'],
     data(){
         return{
             website:{
                 title:'谷歌',
                 url:'https://www.google.com/',
                 subTitle: '全球英文第一搜索引擎'
             }
         }
     }

 }
</script>

```

效果

![image-20211222171605277](C:\Users\Jiali\AppData\Roaming\Typora\typora-user-images\image-20211222171605277.png)

##### 3.3具名插槽

父组件里要插入多部分内容，所以需要不同的slot对应。

```vue
<template v-slot:header>
</template>
```

子组件

```vue
<slot name="header">
```



#### 4.动态异步组件

##### 4.1动态组件

不是很常用，但是也有可能考到。

格式：`:is= "component-name"`

应用场景：比如说渲染新闻页面，有文字，图片，视频等，但不确定究竟是什么类型时，就需要用到动态组件去渲染

 代码演示：

```vue
<template>
    <div>
        <div v-for="(val,key) in newsData" :key="key">
            <!-- 根据类型是什么来渲染 -->
            <!-- 动态组件 component 作为组件名 :is(必须是动态属性) 后面是我们想要动态渲染的组件名 -->
                <component :is="val.type"/>
        </div>
    </div>
</template>
<script>
export default{
    data(){
        return{
             newsData: {
                 // 有不同的 type 所以要渲染这些数据、节点的方法也不尽相同
                 1:{
                     type: "text"
                 },
                2:{
                     type: "vedio"
                 },
                3:{
                     type: "image"
                 }
             }
           
        }
    }
}
 
</script>

```



##### 4.2异步组件 (很常用)

按需加载: 使用`import()` 函数，异步加载大组件

假设有一个比较大的表单组件，名为 `FormDemo`，我们不需要它在一开始就加载出来，而是用户点击时才加载，那么就需要使用异步组件

```vue
<template>
    <div>
        <!-- 异步组件 -->
        <FormDemo v-if="showFormDemo"/>
        <button @click="showFormDemo = true">show FormDemo</button>
    </div>
</template>

<script>
export default{
    components: {
        // 当用户点击时 showFormDemo 才变为 true，此时我们才 import 该组件
       FormDemo: () => import('../BaseUse/FormDemo')
    },
    data(){
        return{
             showFormDemo: false           
        }
    }
}
</script>

```



#### 5.keep-alive

- 缓存组件
- 需要频繁切换但是渲染不变的情况
- `Vue` 常见性能优化

假设有三个需要频繁切换的组件，如果不用 `keep-alive`，切换一次就会渲染一次新组件 并且 `destory` 上一个组件

`KeepAlive`组件 

```vue
// 功能：点击不同按钮显示对应组件
<template>
    <div>
        <button @click="ChangeState('A')">A</button>
        <button @click="ChangeState('B')">B</button>
        <button @click="ChangeState('C')">C</button>       
        <KeepAliveA v-if="state === 'A' "/>
        <KeepAliveB v-if="state === 'B' "/>
        <KeepAliveC v-if="state === 'C' "/>
    </div>
</template>

<script>
import KeepAliveA from './KeepAliveA'
import KeepAliveB from './KeepAliveB'
import KeepAliveC from './KeepAliveC'
 export default{
     components: {
         KeepAliveA,
         KeepAliveB,
         KeepAliveC
     },
     data(){
         return{
             state: 'A'
         }
     },
     methods: {
         ChangeState(state){
             this.state = state
         }
     }
 }
</script>

```

KeepAliveA （其它两个差不多）

```vue
<template>
    <p>A</p>
</template>

<script>
 export default{
     mounted(){
         console.log('A is mounted');
     },
     destroyed(){
         console.log('A is desoryed');
     }
 }
</script>

```

如果我们单纯引入 KeepAliveA、B、C 组件

```vue
	<KeepAliveA v-if="state === 'A' "/>
	<KeepAliveB v-if="state === 'B' "/>
	<KeepAliveC v-if="state === 'C' "/>
```

那么我们切换的时候，组件就会连续销毁旧的渲染新的，影响性能

![image-20211222195008814](C:\Users\Jiali\AppData\Roaming\Typora\typora-user-images\image-20211222195008814.png)

如果包裹在  `<keep-alive>` 标签中，就能一直保持 mounted 的状态

```vue
		<keep-alive>
            <KeepAliveA v-if="state === 'A' "/>
            <KeepAliveB v-if="state === 'B' "/>
            <KeepAliveC v-if="state === 'C' "/>
  		 </keep-alive>   
```



![image-20211222195240112](C:\Users\Jiali\AppData\Roaming\Typora\typora-user-images\image-20211222195240112.png)

#### 6.mixin

##### 6.1优势

- 多个组件有相同逻辑的时候，抽离出来

比如说这个例子，city 属性并没有在当前组件定义，而是引用了抽离出来的 JS 的，而且不仅属性可以抽离，方法也可以

```vue
<template>
    <div>
        <p>{{name}}  {{grade}} {{city}}</p>
    </div>
</template>

<script>
import mixin from './mixin'
 export default{
     // 共同逻辑 JS 文件的注册方法
     mixins:[mixin],
     data(){
         return {
             name: 'zhangsan',
             grade: 'second'
         }
     },
     mounted(){
        console.log('MixinDemo mounted');
    }
 }
</script>

```

mixin.js 文件

```js
export default{
    data(){
        return {
            city: 'beijing'
        }
    },
    mounted(){
        console.log('mixinJs mounted');
    }
}
```

效果 属性和方法逻辑都被读取到了

![image-20211222201903065](C:\Users\Jiali\AppData\Roaming\Typora\typora-user-images\image-20211222201903065.png)

##### 6.2问题

- 来源不明确
- 多 mixin 肯会造成命名冲突
- mixin 和 组件可能造成多对多关系，复杂度变高

#### 7.Vuex

掌握基本概念，基本使用和基本API。

比较常考察 `state` 的设计

`vuex` 是 `vue` 配套的公共数据管理工具，可以将要共享的数据保存到 `vuex` 中，方便整个程序中的任何组件都能够获得和修改公共数据

修改共享数据：在 mutations 中定义

https://vuex.vuejs.org/zh/

https://www.cnblogs.com/qduanxq/p/15099684.html

<img src="../images/vuex.png" style="zoom: 67%;" />

#### 8.vue-router

https://router.vuejs.org/zh/

##### 8.1路由模式

- hash 模式，如 http://abc.com/#/user/10

  原理：url中的#号后面对应的是hash值，通过hashChange() 事件监听hash值的变化，再从路由表中匹配路由。

  优点：1.只要前端配置好路由表即可，不会请求后端，所以不需要后端的参与，完全属于前端；

  ​           2.浏览器兼容性好

  缺点：用#号连接，不符合url规范，不美观

- H5 history 模式 http://abc.com/user/20，需要后端支持

  原理：基于`pushState`和`popstate`实现

  优点：很美观

  缺点：1.前端的URL必须和向发送请求后端URL保持一致，否则会报404错误

  ​           2.由于History API的缘故，低版本浏览器有兼容行问题

- abstract 模式

##### 8.2 动态路由

最简单的模式

```js
const User = {
  template: '<div>User</div>',
}

// 这些都会传递给 `createRouter`
const routes = [
  // 动态字段以冒号开始
  { path: '/users/:id', component: User },
]
```

现在像 `/users/johnny` 和 `/users/jolyne` 这样的 URL 都会映射到同一个路由。

*路径参数* 用冒号 `:` 表示。当一个路由被匹配时，它的 *params* 的值将在每个组件中以 `this.$route.params` 的形式暴露出来。因此，我们可以通过更新 `User` 的模板来呈现当前的用户 ID：

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>',
}
```



##### 8.3懒加载

异步加载,在需要的时候import
