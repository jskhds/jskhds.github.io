---
title: 手写call/apply/bind
date: 2022-03-01 12:41:19
tags: 手写 call apply bind
---

备忘

<!--more -->

#### 1.手写call

##### 原理

```js
Function.prototype.myCall = function(thisArg, ...args){
    console.log(this);
    console.log(thisArg);
}
function fn(){};
fn.myCall('obj','arg1','arg2');  // fn obj
/*
myCall 函数的目标就是把 fn 运行时的 this 指向 obj
换句话说，就是让 fn 成为 obj 这个对象的方法
*/

```

##### 简化版思路

```js
// 简化版 体现思路
Function.prototype.myCall = function(thisArg, ...args){
    // 1.给 thisArg 添加方法 也就是调用 myCall 的函数本身
    thisArg.func = this;
    let result = thisArg.func(...args);
    return result;
}
```

##### 简化版的不足

会出现覆盖问题：如果说我们对传入的 obj 添加的方法都叫 func，那么如果多个函数都调用了这个myCall ，就会存在覆盖的可能

```js
function fn1(){
    console.log(this.x)
}
function fn2(){
    console.log(this.y)
}
const obj = {
    x:'x',
    y:'y'
}
fn1.myCall(obj)  // x
fn2.myCall(obj)  // y
console.log(obj);  // { x: 'x', y: 'y', func: [Function: fn2] } 本来应该既有 fn1 也有 fn2， 现在只有 fn2
```

##### 完善版

加入 Symbol() 构造独一无二的属性名 ，注意 Symbol 不能 new

```js
//完善版
Function.prototype.myCall = function (thisArg, ...args) {
    // 如果 thisArg 是 undefined 或者 null 那么就替换为 全局对象
    thisArg = thisArg || global
    // ES6引入了一种新的原始数据类型Symbol，表示独一无二的值，最大的用法是用来定义对象的唯一属性名。
    const prop = Symbol();
    thisArg[prop] = this;
    let result = thisArg[prop](...args);
    // delete 操作符用于删除对象的某个属性, 此操作为了不更改原来的 obj 
    delete thisArg[prop];
    return result;
  };

function fn1(){
    console.log(this.x)
}
function fn2(){
    console.log(this.y)
}
const obj = {
    x:'x',
    y:'y'
}
fn1.myCall(obj)  // x
fn2.myCall(obj)  // y
console.log(obj); // 如果没有 delete 操作的话，obj 此时就有 fn1 fn2 两个函数
```

#### 2.手写apply

apply 和 call 的实现思路是一样的，只是传参形式不同。call 用参数列表传参，apply 用数组传参

```js
Function.prototype.myApply = function(thisArg, args){
    args = args||[];
    thisArg = thisArg||global;
    const prop =  Symbol();
    thisArg[prop] = this;
    const res = thisArg[prop](...args);
    delete thisArg[prop];
    return res;
}

const obj = {
    x:'x'
}
function fn(x,y,z){
    console.log(this.x)
    console.log(x + y + z)
}
fn.myApply(obj,[1,2,3])
```

#### 3.手写 bind

bind 返回的是一个永久改变了 this 指向的函数，我们可以在 call 或者 apply 中任选一个来改变this 指向

##### 版本一

```js
Function.prototype.myBind = function(thisArg, ...args) {
  const self = this
  // 返回绑定函数
  return function() {
    // 包装了原函数对象
    return self.apply(thisArg, args)
    // or  return self.call(thisArg,...args)
  }
}
```

##### 版本二

```js
Function.prototype.myBind = function() {
    // 把参数处理放到里面来
    const args = Array.prototype.slice.call(arguments)
    const targetT = args.shift()
    const self = this
     return function () {
         return self.apply(targetT,args)
     }
 }
```

