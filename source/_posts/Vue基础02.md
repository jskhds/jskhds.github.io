---
title: Vue基础02
date: 2021-12-21 19:32:35
tags: Vue组件 生命周期
---

Vue 的组件使用以及生命周期

<!-- more -->

#### 1.组件使用

##### 1.1 props + $emit 传递数据

props:父组件传递数据或者函数给子组件

$emit: 命令式地触发该组件实例上的一个自定义事件。在父子组件通信中，子组件使用 $emit 可以修改父组件的属性(不过我感觉这样有点不大适合，因为我可以改你的你可以改我的很容易乱套)

```vue
//子组件 Child.vue
<template>
    <p>{{message}}</p>
    <button @click="handleClick">点击修改数据</button>
</template>

<script>
export default{
    name:'child',
    // 使用父组件传递过来的 props
    props:[
        'message',
    ],
    methods:{
        // 触发事件，修改父组件的属性
        handleClick(){
            // this.$emit('父组件被触发函数的函数名，用@' ,'传递的参数')
            this.$emit('ChangeMessage','Hello Again')
        }
    }
}
</script>

// 父组件 
<template>
    <p>{{message}}</p>
    <child :message="message" @ChangeMessage="ChangeMessage"/>
</template>

<script>
import Child from "./Child.vue";
export default{
    name:'parent',
    components:{
        Child,
    },
    data(){
        return {
            message:"Hello World"
        }
    },
    methods:{
        ChangeMessage(msg){
            this.message = msg;
        }
    }
}
</script>
```



##### 1.2 eventBus

新建一个js文件，导出一个 vue 实例

```js
import Vue from 'vue'
export default new Vue()
```

使用 eventBus.$emit, eventBus.on

##### 1.3 组件生命周期（要自己画出来）

<img src="../images/vue_lifecycle.svg" style="zoom: 50%;" />

- 挂载阶段

到 `mounted`  问：`created` 和 `mounted` 有什么区别？

- 更新阶段

到 	`updated`

- 销毁阶段

最后两个



##### 1.4 带有父子组件的生命周期

1.加载渲染过程
 父`beforeCreate`->父`created`->父`beforeMount`->子`beforeCreate`->子`created`->子`beforeMount`- >子`mounted`->父`mounted`

2.子组件更新过程
父`beforeUpdate`->子`beforeUpdate`->子`updated`->父`updated`

3.父组件更新过程
父`beforeUpdate`->父`updated`

4.销毁过程
父`beforeUnmount`->子`beforeUnmount`->子`unmounted`->父`unmounted`

PS：之前的 `beforeDestory` 和`destoryed` 对应`beforeUnmount` 和 `unmounted`





