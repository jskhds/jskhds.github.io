---
title: JS面试篇-Part1
date: 2021-11-28 16:35:03
tags: 值类型 引用类型 深拷贝 ==和=== truly和falsely变量
---

#### 1.值类型和引用类型以及区别

 <!-- more -->

##### 1.1 定义

- 值类型，value直接存储在栈中，值的地址互不干扰

```js
let a = 100;
let b = a;
a = 200;
console.log(a); //200
console.log(b); //100
```

- 引用类型，元素指向地址，值存放在元素指向的地址中

```js
let a = { age: 20 };
let b = a;
b.age = 21;
console.log(a.age); //21 a 和 b 指向的地址都是一样的，所以改了 b 后 a 也会改
```

- 引用类型比值类型占用的内存要大得多，如果直接复制值，资源浪费大，并且复制时间长。

##### 1.2 常见值类型

- 基本数据类型存储在栈内存，存储的是值。
- 复杂数据类型的值存储在堆内存，地址（指向堆中的值）存储在栈内存。（堆存地址，栈存值）

`Null Number Undefined Symol String Boolead`

```js
let a //undefined
const a // 会报错,const 必须赋值
const s = 'abc'
const n = 100
const b = true
const sym = Symbol('s')
```

##### 1.3常见引用类型

```js
const obj = { x: 100 }
const arr = ['a', 'b', 'c']
const n = null //特殊引用类型，指针指向空地址，其实也算基本数据类型，也算特殊引用类型
function fn() {} //特殊引用类型，不用于存储数据，所以不存在 深拷贝，浅拷贝 的概念，也可以看做第三个类型
```

##### 1.4 值类型和引用类型例题

```js
const obj1 ={
    x:100,
    y:200
} 
const obj2 = obj1
let x1 = obj1.x
x1 = 102 // 干扰项，x1 是值类型，直接赋值，它的改变不会影响其它的量
obj2.x = 101
console.log(obj1.x); // 101
```



##### 1.5 typeof 可以判断的类型

1. 识别所有值类型
2. 识别函数
3. 判断是否为引用类型（判断出是 `object` 就没了，不再细分）

```js
let a                	//typeof(a) undefined 
const s = 'abc'      	//typeof(s) string
const n = 100 			//typeof(n) number
const b = true 			//typeof(b) boolean
const sym = Symbol('s')  //typeof(sym) symbol
typeof(console.log)		 //function
// 其余引用类型 typeof 只能判断为 object

```

#### 2.变量计算-类型转换

##### 2.1 字符串拼接

```js
const a = 100 + 10 //110
const b = 100 + '10' //'10010'
const c = true + '10' //true10
// 强制类型转换
const d = 100 + parseInt('100'); //200
```

##### 2.2 ==

何时使用 `== `何时使用 ` ===`

- == 尽量转换 让二者相等，也就是把二者的类型转换成一样的再比较值是否相等

```js
// 用 == 基本都是true，=== 才会返回false
100 == '100' //true
0 == '' //true
0 == false //true
false == '' //true
null == undefined //true
```

- 除了 == null 以外，其他一律都用 ===

##### 2.3 if语句和逻辑运算

- truly 和 falsely 变量

​	if 语句里判断的不是 true 或者 false 而是把变量 !! 之后判断是 truely 还是 falsely

```js
// truly 变量
!!a === true
// falsely 变量
!!b === false

// 以下是 falsely 变量，其余都是 truly 变量
!!0 === false
!!NaN === false
!!'' === false
!!null === false
!!undefined === false
!!false === false
```

- 在 if 语句中的应用（if判断 !! 之后是 truly 变量还是 falsely 变量） 

```js
// truly 变量
    const a = true 
    if (a){
    	...
    }
    const b = 100
    if(b){
    	...
    }
```

```js
// falsely 变量
    const c = ''
    if(c){
    	...
    }
    const d = null
    if(d){
    	...
    }
    let e
    if(e){
   		 ...
    }
    else{
         //会走到这一步
    }
    
```

- 逻辑判断

```js
console.log(10 && 0) // 0
console.log('' || 'abc') // 'abc'
console.log(window.abc) //false
```

 

##### 2.4 手写深拷贝

- 注意判断值类型和引用类型
- 注意判断数组和对象
- 递归

```js
// 深拷贝部分
 
function deepClone(obj){
    let res = obj instanceof Array?[]:{};
    if(typeof(obj)!=='object' || typeof(obj) == null){
        return obj;
    }
    for(let key in obj){
        // 确保key不是原型链上的而是自身可以迭代遍历的属性
        // hasOwnProperty() 方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性(也就是是否有指定的键)
        if(obj.hasOwnProperty(key)){
            // 递归（重点） key要一层一层遍历，比如说 obj{ address:{city: 'Beijing'}}
            res[key] = deepClone(obj[key])
        }
    }   
    return res;
}


// test 部分
let obj1 = {
    address:{
        city:"Beijing"
    }
}

let obj2 = deepClone(obj1);
obj2.address.city = "Shanghai";
console.log(obj1.address.city);
console.log(obj2.address.city);
```

Q：深拷贝的堆栈模型图？

栈内存址，堆内存值，深拷贝后地址和指向都是分开的
