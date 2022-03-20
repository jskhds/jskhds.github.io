---
title: 前端面试真题-part1
date: 2021-12-15 13:57:04
tags: 前端面试
---

主要内容：var let const 、深度比较、数组的方法

<!-- more -->

#### 1. part-1

##### 1）var let const 的区别

- var 是 ES5 语法，let const 是 ES6 语法；var 有变量提升

```js
// ES5 变量提升
// ES5在执行的时候，会先把 var 定义的值先提出来定义为 undefined
console.log(a); // undefiend
var a = 200

// 相当于 
var a
console.log(a); // undefiend
a = 200
```

- var 和 let 是变量，可以修改，const 是常量， 不可以修改
- let 和 const 都有块级作用域，var 没有

```js
// var 定义的变量是全局变量
for(var i = 0;i<10;++i){
    var j = i + 1
}
console.log(i,j);   // 10

// let 定义的变量只在自己的块级作用域里
// 块级作用域
for(let i = 0;i<10;++i){
    let j = i + 1
}
console.log(i,j); // 报错 undefined

```

##### 2）typeof 返回

- undefined string number boolean symbol （值类型）
- object （注意 typeof null === 'object' 判断为 true）（null 本质上相当于一个空指针）
- function （函数是object 但可以被判断出来）

##### 3）列举 强制类型转换 和 隐式类型 转换

- 强制：parseInt parseFloat toString
- 隐式：if、逻辑运算、==、+ 拼接字符串



#### 2.part-2

##### 1）手写深度比较，模拟 lodash isEqual

```js
// 另写一个函数 判断是否为 不为空的对象或数组
function isObject(obj){
    return typeof obj === 'object' && obj !== null
}
// 递归函数
function isEqual(obj1,obj2){
    // 递归终止
    if(!isObject(obj1)||!isObject(obj2)){
        return obj1 === obj2
    }
    if(obj1 === obj2){
        return true
    }
    // 确保两个都是对象或数组
    const obj1Keys = Object.keys(obj1)
    const obj2Keys = Object.keys(obj2)
    if(obj1Keys.length !== obj2Keys.length){
        return false
    }
    //  长度相等的时候，递归比较
    for(let key in obj1){
        const res = isEqual(obj1[key],obj2[key])
        if(!res){
            return false
        }
    }
    
    return true
}
```

##### 2）split() 和 join() 的区别

```js
'1-2-3'.split('-') // [1,2,3]
[1,2,3].join('-') // '1-2-3'
```

##### 3）数组的 pop push unshift shift 分别做什么

从三个方面来说

- 分别的功能
- 分别的返回值
- 是否会对原数组造成影响

```js
// pop  删除最后一个元素，返回删除的这个元素，会改变原数组
let a = [1,2,3]
console.log(a.pop(),a);  // 3, [1,2] 
 
// shift 删除第一个元素，返回删除的这个元素，会改变原数组
let b = [4,5,6]
console.log(b.shift(),b); // 4, [5,6] 
 
// push 把元素加到最后面,返回数组的长度，会改变原数组
let c = [7,8,9]
console.log(c.push(10),c);  // 4 [ 7, 8, 9, 10 ]  
 
// unshift 把元素加到最前面,返回数组的长度，会改变原数组
let d = [11,12,13]
console.log(d.unshift(14),d);  // 4 [ 14, 11, 12, 13 ]  
```



##### 4）数组的纯函数 API

 纯函数的定义

1.不改变原数组；2.返回一个数组。 

所以 3）中的 API 都不是纯函数，还有 forEach，some every，reduce，这三个虽然不改变原数组，但不返回一个数组

以下列举一些常见的数组纯函数 API：concat map filter slice

```js
//  concat map filter slice 都不会改变原数组 同时返回一个新数组
const arr = [1,2,3,4]
const arr1 = arr.concat([5,6,7,8]) // 拼接两个数组 [1, 2, 3, 4, 5, 6, 7, 8]
const arr2 = arr.map( num => num * 3) // 按照一定规则遍历改变元素  [ 3, 6, 9, 12 ]
const arr3 = arr.filter( num => num > 2) // 按照一定规则过滤元素 [ 3, 4 ]
const arr4 = arr.slice() // 相当于深拷贝原数组 [ 1, 2, 3, 4 ]
```



#### 3.part3

##### 1）数组 slice 和 splice 的区别

从三个方面来说

- 功能区别
- 参数和返回值
- 是不是纯函数

```js
const arr = [1, 2, 3, 4]
// slice 纯函数
const arr1 = arr.slice()  // 深拷贝  [1, 2, 3, 4]
const arr2 = arr.slice(1,3) // slice(startIndex,endIndex) 左闭右开 [2, 3]
const arr3 = arr.slice(1)  // slice(startIndex) 从开始截取到最后 [2, 3, 4]
const arr4 = arr.slice(-2) // slice(负值) 倒数的几个值 [3, 4]

// splice 非纯函数 三个参数 (startIndex, length, replaceValue) 后两者都是可选的，如果不写 replaceValue相当于删去原数组的元素
const arr5 = arr.splice(1,2,'a','b','c')
console.log(arr5); // [2, 3]
console.log(arr); // [ 1, 'a', 'b', 'c', 4 ]
```

##### 2）[10, 20, 30].map(parseInt)

- map 的参数和返回值 (item, index)  返回一个新数组
- parseInt 的参数和返回值 parseInt(string, radix);  解析一个字符串并返回指定基数的十进制整数，也就是说 把一个 radix 进制的 string 转换为一个十进制数并且输出

```js
const res = [10,20,30].map(parseInt)
console.log(res);  // [ 10, NaN, NaN ]

// 拆解来看： 
res = [10, 20, 30].map((item,index)=>{
    // parseInt(10,0); 0 相当于 10 进制，也就是把一个 10 进制的数变成 10 进制输出
    // parseInt(20,1); 把一个 1 进制的数 20 输出为 10 进制，给定的 20 不符合 1 进制 所以输出 NaN
    return parseInt(item,index)
})
```

##### 3）ajax 的请求 get 和 post 的区别

- get 一般用于查询操作，post 一般用户提交操作
- get 参数拼接在 url 上， post 放在请求体内（数据体积更大）
- 安全性：post 防止 CSRF/XSS攻击

##### 4)get 和 post 的区别

网上又有人说其实本质上没什么区别，我仔细阅读之后感觉说得有道理，但是还是要知道一些普遍答案吧、

1. GET使用URL或Cookie传参。而POST将数据放在BODY中。

2. GET的URL会有长度上的限制，则POST的数据则可以非常大。

3. POST比GET安全，因为数据在地址栏上不可见。

4. 并不是所有浏览器都会在POST中发送两次包，Firefox就只发送一次。

















