---
title: 前端面试真题-part5
date: 2021-12-15 22:05:42
tags: 
---

主要内容：Map Set 有序和无序 数组reduce求和 

<!-- more -->

#### 1-part1

##### 1） Map 和 Set 有序和无序

-  有序慢，无序快。但是无序本身就是一个缺点。比如说 v-dom里面，object什么的可以无序排列，但是数组必须是有序的。不然渲染过程会出现问题
- 可以结合二者优点：二叉树及其变种

##### 2）Map 和 Object 区别

- API 不同（略过，比较简单），Map可以以任意类型为 key

```js
function fn()
const obj = {}
let m = new Map()
m.set(fn,'fn key') 
m.set(obj,'obj key')
// 如果说是 Object
obj.fn() ='XXX' h// 报错 object 只能以 字符串为 key
```



- Map 是有序结构（重要）

```js
let map = new Map([
    ['key1','1'],
    ['key2','2'],
    ['key3','3']
    ])

    map.forEach((value,key)=>{
        console.log(key,value)
})  
// key1 1
// key2 2
// key3 3
// 如果把 map 的顺序改了 输出的顺序也会改
```



- Map 操作同样很快 对比无序结构object 的 查找和删除操作 map 甚至会更快一点。

```js
const obj = {}
for(let i = 0;i<1000*1000;i++){
    obj[i+''] = i
}
console.time('obj find');
obj['500000']
console.timeEnd('obj find');
console.time('obj delete');
delete obj['500000']
console.timeEnd('obj delete');

const map = new Map()
for(let i = 0;i<1000*1000;i++){
    map.set(i+'',i)
}
console.time('map find');
map.has('500000')
console.timeEnd('map find');
console.time('map delete');
map.delete('500000')
console.timeEnd('map delete');
```



##### 3）Set 和 数组的区别

- API 不同

```js
const set = new Set([10,20,30])
set.add(50)
set.delete(20)
set.has(30)
set.size
ser.forEach(val=> console.log(val)) // 无序结构 没有index
```

- Set 元素不能重复
- Set 是无序结构，操作很快，和上面的代码差不多

```js
 // 数组比较慢 尤其是查找
let arr = [10,20,30]
for(let i = 0;i<1000*1000;i++){
   arr[i] = i
}
console.time('arr push');
arr.push('a')
console.timeEnd('arr push');
console.time('arr unshift');
arr.unshift('b')
console.timeEnd('arr unshift');
console.time('arr includes');
arr.includes(100000)
console.timeEnd('arr includes');
```

```js
// set 就快很多
let set = new Set([10,20,30])
for(let i = 0;i<1000*1000;i++){
   set.add(i)
}
console.time('set add');
set.add('a')
console.timeEnd('set add');
 
console.time('set has');
set.has(100000)
console.timeEnd('set has');
```

##### 4）WeakMap 和 WeakSet

- 弱引用，防止内存泄漏

​       可以随意添加信息，但是执行完了之后，对象就被销毁了，不用考虑是否还需要引用这些对象

- WeakMap 只能用对象作为 key ，WeakSet 只能用 对象作为 value
- 没有 forEach 和 size ，只能用 add has delete



##### 5）数组求和- Array reduce

```js
let arr = [1,2,3,4]
// 比较简单易读，但是会多一个变量
function sum(arr){
    let res = 0
    for(let i = 0;i<arr.length;i++){
        res += arr[i]
    }
    return res
}
console.log(sum(arr));
```

Array reduce 数组的拼接 计数什么的都很好用

```js
let arr = [1,2,3,4]
const sum = arr.reduce((sum,curVal,index,arr)=>{
    console.log('reduce function');
    console.log('sum',sum);
    console.log('curVal',curVal);
    console.log('index',index);
    console.log('arr',arr);
    return sum + curVal
},0)
 
console.log('final sum',sum);

```

