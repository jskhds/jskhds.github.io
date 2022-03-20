---
title: JS面试篇-Part3
date: 2021-11-30 20:32:38
tags: 作用域 闭包 this 手写bind
---

#### 1.作用域

<!-- more -->

- 全局作用域

- 函数作用域

- 块级作用域

  - ```js
    // 在 if for while 这些语句的作用域叫块级作用域
    if(true){
    	let x = 100
    }
    console.log(x) // 报错
    ```

  

  - 代码演示

  ```js
  // 函数作用域，到自己定义处的上级一级一级找
  
  function fn1(){
      let a = 100
      fn2()
      function fn2(){  
          let a1 = 200 
          function fn3(){
              let a2 = 300
              return a + a1 + a2
          }
          return fn3()
      }
     return  fn2()
  }
  console.log(fn1())  // 600
  /*
  	说明：fn1 的返回值是 fn2(),所以去找 fn2
  	fn2 的返回值是 fn3() ,所以去找 fn3,
  	fn3 的返回值是 a + a1 + a2 所以最终 fn1 的返回值是 a + a1 + a2
  */
  ```
  
  

#### 2.自由变量

- 在本级作用域使用但是没有定义的变量
- 一级一级向上找，直到找到为止
- 如果找不到则报错 xx is not defined

#### 3.闭包

一个函数和其周围状态（**lexical environment，词法环境**）的引用捆绑在一起（或者说函数被引用包围），这样的组合就是**闭包**（**closure**）。也就是说，闭包让你可以在一个内层函数中访问到其外层函数的作用域。在 JavaScript 中，每当创建一个函数，闭包就会在函数创建的同时被创建出来。

- 闭包：（所有的）自由变量查找，**是在函数定义的地方向上查找**，不是在函数执行的地方！！！
- 作用域应用的特殊情况有以下两种
- 函数作为返回值

```js
// 1.函数作为返回值
function create(){
    const a = 100
    return function (){
        console.log(a)
    }
}
//  因为返回的是一个函数 如果要执行这个返回的函数 需要拿一个函数接着
create()() // 100 这样写也行，但显得不是很正规
const createExecu = create()
createExecu() // 100
const a = 200
createExecu() // 100
let b = 200
createExecu(b)  // 100

//2.返回
function fn() {
    let a = 1;
    return function fn1(){
        return a;
    }
    
}

console.log(fn());
console.log(fn()());
// [Function: fn1]
// 1
```



- 函数作为参数

```js
function print(fn){
    const a = 200
    fn()
}
const a = 100
function fn(){
    console.log(a);
}
print(fn) // 100
```



#### 4.this

**this 在函数执行的时候才确定值**，不是在函数定义的时候确认的

##### 4.1调用场景

- 作为普通函数 返回window
- 使用 call apply bind

```js
function fn1(){
    console.log(this);
}
fn1() // window
// call 和 bind 可以改变函数 this 的指向 但是用法有所不同
fn1.call( {x:100} )  // { x: 100 }
// bind 返回一个新的函数执行
const fn2 = fn1.bind( {x:200} ) 
fn2() // { x: 200 }
```



- 作为对象方法被调用 setTImeout里面用普通函数，this 指向 window

```js
const zhangsan = {
    name: "张三",
    sayHi(){
        console.log(this);
    },
    wait(){
        setTimeout(function(){
            console.log(this);
        }) 
    }
}
zhangsan.sayHi() // 当前对象 也就是 zhangsan 这个对象
zhangsan.wait() // setTimeout  里面的普通函数 this 指向 window
```



- 箭头函数 setTimeout里是箭头函数 指向上一级this

```js
const zhangsan = {
     waitAgain(){
         setTimeout(()=>{
             console.log(this);
         })
     }    
 }
 zhangsan.waitAgain()  // waitAgain 这个函数
```



- class方法中调用

```js
class People{
	constructor(name,number){
	this.name = name
	this.number = number
	}
	sayHi(){
	console.log(this)
	}
}
const zhangsan = new People('张三',40)
zhangsan.sayHi() // this 指向 zhangsan
```

#### 5.题目

##### 5.1.this的不同应用场景

​	4.1就是答案

​	情况很多，需要重复记忆

##### 5.2.手写bind函数

​	bind 传入一个this 返回一个函数

​	需要注意的是箭头函数没有自己的 this , 所以不能用 bind 动态修改 this 的指向

- 注释分析版

```js

function fn1(a,b,c){
    console.log("this",this);  // 单纯的this 就是window
    console.log( a,b,c)
    return "this is fn1"
}
 
// const fn2 = fn1.bind({x:100},1,2,3) //bind改变函数this的指向
// fn2()  //this { x: 100 }

// 原型链分析
// let flag = fn1.hasOwnProperty("bind")
// console.log(flag) // false
// flag = fn1.__proto__===Function.prototype?true:false // true
// console.log(flag);

 Function.prototype.bind1 = function() {
    //  先把传进来的参数列表变成数组 arguments 可以获得所用的参数

    const args = Array.prototype.slice.call(arguments)
    // 用shift 获取 args 的第一项也就是传进来的this（shift会改变原数组）
    const t = args.shift()
    // 原本的this 也就是 fn1 拿出来
    const self = this
    // 返回一个函数（实现bind的功能）
     return function () {
        //  apply 第一个参数是 this 第二个参数是 参数数组
         return self.apply(t,args) 
     }
 }

const fn3 = fn1.bind1({x:100},1,2,3) //bind1改变函数this的指向
fn3()  //this { x: 100 }
```

- 答案版

```js
Function.prototype.bind1 = function() {
    const args = Array.prototype.slice.call(arguments)
    const t = args.shift()
    const self = this
     return function () {
         return self.apply(t,args)
     }
 }
```

补充：类数组转换为数组

```js
// Array.prototype.slice.call('参数列表','要截取的部分(可选)') => 真正的数组
const args = Array.prototype.slice.call(arguments) 
// or
const args = [...arguments] 
// or
const args = [];
for(let i = 0;i<arguments.lenght;i++){
    args[i] = arguments[i]
}
```



##### 5.3.实际开发中闭包的应用场景，举例说明

- 隐藏数据 不让外部改变处理数据的方法
- 做一个简单的 cache 工具

```js
function createCache() {
    let data = {}
    return {
        // set: function(key,value){} 格式也是对的
        set(key,value){
            data[key] = value
        },
        get(key){
            return data[key]
        }
    }
    
}
const fn1 = createCache()
fn1.set("age",18)
console.log(fn1.get("age"));

```



##### 5.4 创建10个 a 标签点击的时候弹出相应的序号

```js
   // 创建10个 a 标签，点击弹出相应索引 
   // 考点在于 如果 i 在 外部声明而不是for循环里面声明 那么最后i的值就一直是 10 
   // 因为遍历创建 a 标签的速度很快 i又是全局变量 所以一下就加完了，但是事件不是同步任务，需要点击才能触发，所以当我们点击 a 标签的时候，i 已经变成 10 了 只有把 i 放到块级作用域里，才能和alert事件同步。因为每遍历一次，i会开辟一个新的作用域，在alert的时候，就在对应作用域里找 i 是多少 
   let a 
   for(let i = 0;i<10;i++){
       a = document.createElement('a')
       a.innerHTML = i + '<br>'
       a.addEventListener('click',function(e){
           e.preventDefault()  // 阻止点击 a 标签跳转的默认事件
           alert(i)
       })
       document.body.appendChild(a)
   }
```

