---
title: 前端面试真题-part3
date: 2021-12-15 19:13:57
tags: 
---

主要内容：函数声明和函数表达式、new Obeject() 和 Object.create()、 this 

<!-- more -->

#### 1.part1

##### 1）函数声明和函数表达式的区别

- 函数声明 function fn(){...}
- 函数表达式 const fn = function() {...}
- 函数声明会在代码执行前预加载，但是函数表达式不会

```js
// 函数声明 代码执行前函数已经初始化好了，所以不会报错
const res = sum(10,20) 
console.log(res)
function sum(x, y){
	return x +y
}  

// 函数表达式，用 let const 声明，不会提升所以函数就不会提前初始化，会报错
// 用 var 定义，则 sum 已经变量提升，执行的时候 sum 是 undefined，代码会报错 sum  is not a function
const res = min(10,20)
console.log(res)
const min => (a,b){
	return a - b
}

```

##### 2）new Obeject() 和 Object.create() 的区别

- new Object() 和 {} 是等同的，所以它们的原型都是 Object.prototype
- Object.create(null) 没有原型（）里的是要传给其的参数
- Object.create({...}) 可以指定原型

```js
const obj1 = {
    a: 10,
    b: 20,
    sum(){
        return this.a + this.b
    }
}

const obj2 = new Object(obj1) // obj1 === obj2
const obj3 = Object.create(null) // 没有属性 没有原型
const obj4 = new Object(null)  // 没有属性 但是有原型
const obj5 = Object.create(obj1) // 此时 obj5 的隐式原型是 obj1 也就是 Object，所以 obj5 的属性是空的，但是可以读取原型上的属性和方法
const flag = (obj1 === obj5) // false
const flag1 = (obj5.__proto__ === obj1)  // true
```

##### 3）关于 this 的场景题

原则：函数里面 this 的值在执行的时候才知道，

```js
const User = {
    count: 1,
    getCount: function(){
        return this.count
    }

}

const a = {
    count: 2
}
console.log(User.getCount()); // 1 ，this 就是 User， this.count 就是 User.count
const func = User.getCount
// 如果把 User的 getCount 作为单独的函数执行，那么this 就相当于 window了
console.log(func()); // undefined  
console.log(func.call(a)); // 执行的时候 this 指向 a 就有 count 了


```



#### 2.part2

##### 1）关于作用域和自由变量的场景题-1

一是全局变量，二是 setTimeout 是异步函数

对于全局变量，由于setTimeout 是异步函数，循环是同步，所以要等同步函数执行完，但是这样全局变量已经加到 4 了

```js
let i
for(i = 1; i<=3; i++){
	setTimeout(()=>{
		console.log(i)
	},0)
}
// 4 4 4

for(let i = 1;i<=3;i++){
    setTimeout(() => {
        console.log(i);
    }, 0);
}
// 1 2 3

let j 
for(j = 1;j<=3;j++){
	console.log(j)
}
console.log(j)
// 1 2 3 4 对于同步函数 一次循环从上往下执行 所以即使是全局变量 也是一个挨一个打印
```



##### 2）关于作用域和自由变量的场景题-2

```js
let a = 100
function test(){
	alert(a)
	a = 10
	alert(a)
}
test()
alert(a)
// 100 10 10
```

##### 3）判断字符串以字母开头，后面字母数字下划线，长度 6-30

原则：利用正则表达式（一般来说面试的题比较简单，日常工作还是需要查的）

解释：/ $/ 正则表达式的开头结尾（可选，看是要全部满足还是满足开头或者结尾）；^ 表示字符串开始；[ ] 里的内容表示选择规则，也就是 a-zA-Z的字母保留 \w 表示 匹配字母数字下划线，{5,29} 表示长度 是 >= 5 <=29 (因为前面 \w 已经占了一个长度) 

```js
const reg = /^[a-zA-Z]\w{5,29}$/
```

```js
//基础的正则表达式
// 邮政编码
/\d{6}/
// 英文小写字母
/^[a-z]+$/
// 英文字母
/^[a-zA-Z]+$/
// 日期格式
/^\d{4}-\d{1,2}-\d{1,2}$/
// 简单的IP地址
/\d+\.\d+\.\d+/

```



常见正则表达式 https://deerchao.cn/tutorials/regex/regex.htm   https://www.runoob.com/regexp/regexp-tutorial.html



#### 3.part3

##### 1）手写字符串 trim（把开头和结尾的空格删掉） 方法，保证浏览器兼容性

原则：用 正则表达式 来解决，同时也有原型和this

```js
if(!String.prototype.trim){
	String.prototype.trim = function(){
    return this.replace(/^\s+/,'').replace(/\s+$/,'')
}}
// replace(/^\s+/,'')选中开头是多个（+号的作用）空格的字符串，把多个空格用 '' 代替
// replace(/\s+$/,'')选中结尾是多个（+号的作用）空格的字符串，把多个空格用 '' 代替
// 因为 trim 是 ES5 的，可能有的不支持，所以先判断 String 的原型上有没有 trim 方法，没有就添加
// this 通过 字符串.prototype 执行，this 就是这个字符串
```

##### 2）如何获取多个数组中的最大值

solution 1

```
Math.max()
Math.min()
```

solution 2 

```js
function findMax  (){
    const nums = Array.prototype.slice.call(arguments)

    let max = nums[0]
    console.log(nums);
    nums.forEach(i => {
        if(max <= i )
        max = i
    });
    return max
}
```

##### 3）如何用 JS 实现继承

- class 继承（更加推荐）
- prototype 继承（不推荐）





​			



