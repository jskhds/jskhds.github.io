---
title: JS面试篇-Part6
date: 2021-12-09 14:33:35
tags:  微任务 宏任务 eventloop和dom渲染 手写promise
---

#### 1.宏任务 macroTask 和 微任务 microTask

- 宏任务： setTimeout，setInterval，Ajax，DOM事件
- 微任务：Promise async/await

<!-- more -->

```js
console.log(100); // 1

setTimeout(()=>{
    console.log(200);}) // 4

Promise.resolve().then(()=>{
        console.log(300); // 3
})

console.log(400); // 2
```



#### 2. eventloop和DOM渲染

- eventloop里面 call Stack 空的时候，DOM 开始渲染
- 微任务比宏任务执行时间要早，在DOM渲染前触发
- 宏任务在DOM渲染完成之后触发
- 代码示例：promise→DOM渲染→setTImeout

```js
const $p1 = $('<p>一段文字</p>')
const $p2 = $('<p>一段文字</p>')
const $p3 = $('<p>一段文字</p>')
$('#container')
    .append($p1)
    .append($p2)
    .append($p3)

Promise.resolve().then(()=>{
    console.log('length1',
    $('#container').children().length);
    alert('Promise then')
}) // DOM渲染前触发

setTimeout(()=>{
    console.log('length2',
    $('#container').children().length);
    alert('setTimeout')
}) // DOM渲染后触发
```



#### 3.微任务执行更早的原因

- 微任务是ES6语法规定的，不会经过Web APIs

- 宏任务是由浏览器规定的，按照eventloop常规流程（浏览器→call stack→WebAPIs（等待时机）→进入Callback QUeue→触发eventloop）

- 总的来说：1. Call Stack清空（同步代码都执行完）

  ​                   2.执行当前的微任务（micro task stack）

  ​					3.尝试DOM渲染

  ​					4.触发eventloop（宏任务）

#### 4.问题

##### 4.1 描述eventloop流程（可画图）

A:回顾eventloop的过程即可，不需要一下子就把DOM渲染还有微任务宏任务这些一起说出来，容易乱

##### 4.2 宏任务和微任务，二者区别

A: 二者分类，区别就是执行时机，微任务在DOM渲染前，宏任务在DOM渲染后

##### 4.3 promise的三种状态，变化

A: pending resolved rejected；pending可以变成resolve也可以变成rejected，不可逆；resolve可以触发 then，reject可以触发 catch

##### 4.4 promise场景题 part5 里的第 4点

##### 4.5 async/await场景题

- 执行async 返回的promise 不管里面是什么
- 执行 await 后面的相当于 then

```js
async function fn(){
    return 100
}
   (async function (){
    const a = fn() // async 返回的是一个 promise 对象
    const b = await fn() // await 相当于 then，可以拿到 return 的数据
    console.log(a); // Promise { 100 }
    console.log(b); // 100
   })()
```

```js
(async function(){
    console.log('start');
    const a = await 100 // 相当于返回 100
    console.log('a',a);
    const b = await Promise.resolve(200)
    console.log('b',b);
    const c = await Promise.reject(200) // await 相当于then 只会被resolve状态触发，只能拿到then返回的数据。所以rejected状态以后的所有代码都不会被执行
    console.log('c',c);
    console.log('end');
})()
// 依次打印  start a 100 b 200 
```

##### 4.6 promise和setTimeout的顺序

```js
console.log(100); // 1

setTimeout(()=>{
    console.log(200);}) // 4

Promise.resolve().then(()=>{
        console.log(300); // 3
})

console.log(400); // 2

// 依次打印 100 400 300 200
```

##### 4.6 外加 async/await 的顺序问题

```js
// 先执行同步，call stack清空，执行微任务，DOM渲染，再执行宏任务
async function async1(){
    console.log('async1 start'); // 2.立即执行async里的函数体
    await async2() // 先执行 async2 再执行 await，await后面的都是异步回调 作为微任务
    console.log('async1 end'); // 6 执行微任务 打印async1 end（如果前面没有await 则也一样立即执行）
}
   
async function async2(){
    console.log('async2'); // 3.立即执行 async里面的函数体
}

console.log('script start'); // 1.打印 script start
setTimeout(()=>{
    console.log('setTimeout');
},0) // 8.宏任务 最后执行

async1() // 顺序执行 async1 的函数体 执行到3 async1 里面的同步代码就执行完了 继续往下走
new Promise(function(resolve){
    console.log('promise1'); // 4.promise里面的function立即执行
    resolve(); // 返回resolve状态 后面的then就是回调了 作为微任务
}).then(function(){
    console.log('promise2'); //7. 执行微任务 打印 promise2
})

console.log('script end'); // 5. 到这里整个函数的同步任务就执行完了 开始执行微任务

// script start
// async1 start
// async2
// promise1
// script end
// async1 end
// promise2
// setTimeout
```

#### 5.手写promise

##### 5.1要求

1. 初始化，异步调用

```js
// 初始化
const p = newMyPromise((resolve,reject)=>{
    // resolve(data) 同步调用
    setTimeout(()=>{
        resolve(data) // 异步调用
    })
})
```

​    2. then catch 链式调用

```js
p.then(data=>{
	return data + 1
}).then(data =>{
	return data + 2
}).catch(err =>{
	console.error(err)
})
// 拆开写，只是为了表述出 then catch 返回出的是一个新的promise
const p11 = p.then(data =>{
	return data + 1
})
const p12 = p11.then(data =>{
	return data + 2
})
const p13 = p12.catch(err =>{
	console.error(err)
})

```

3. 支持常见API： .resolve .reject .all .race

```js
const p2 = MyPromise.resolve(200)
const p3 = MyPromise.reject('错误信息...')
const p4 = MyPromise.all([p1,p2])
const p5 = MyPromise.race([p1,p2])
```

##### 5.2 代码实现 （By双越老师）

```js
/**
/**
 * @description MyPromise
 */

 class MyPromise {
    state = 'pending' // 状态，'pending' 'fulfilled' 'rejected'
    value = undefined // 成功后的值
    reason = undefined // 失败后的原因

    resolveCallbacks = [] // pending 状态下，存储成功的回调
    rejectCallbacks = [] // pending 状态下，存储失败的回调

    constructor(fn) {
        const resolveHandler = (value) => {
            if (this.state === 'pending') {
                this.state = 'fulfilled'
                this.value = value
                this.resolveCallbacks.forEach(fn => fn(this.value))
            }
        }

        const rejectHandler = (reason) => {
            if (this.state === 'pending') {
                this.state = 'rejected'
                this.reason = reason
                this.rejectCallbacks.forEach(fn => fn(this.reason))
            }
        }

        try {
            fn(resolveHandler, rejectHandler)
        } catch (err) {
            rejectHandler(err)
        }
    }

    then(fn1, fn2) {
        fn1 = typeof fn1 === 'function' ? fn1 : (v) => v
        fn2 = typeof fn2 === 'function' ? fn2 : (e) => e

        if (this.state === 'pending') {
            const p1 = new MyPromise((resolve, reject) => {
                this.resolveCallbacks.push(() => {
                    try {
                        const newValue = fn1(this.value)
                        resolve(newValue)
                    } catch (err) {
                        reject(err)
                    }
                })

                this.rejectCallbacks.push(() => {
                    try {
                        const newReason = fn2(this.reason)
                        reject(newReason)
                    } catch (err) {
                        reject(err)
                    }
                })
            })
            return p1
        }

        if (this.state === 'fulfilled') {
            const p1 = new MyPromise((resolve, reject) => {
                try {
                    const newValue = fn1(this.value)
                    resolve(newValue)
                } catch (err) {
                    reject(err)
                }
            })
            return p1
        }

        if (this.state === 'rejected') {
            const p1 = new MyPromise((resolve, reject) => {
                try {
                    const newReason = fn2(this.reason)
                    reject(newReason)
                } catch (err) {
                    reject(err)
                }
            })
            return p1
        }
    }

    // 就是 then 的一个语法糖，简单模式
    catch(fn) {
        return this.then(null, fn)
    }
}

MyPromise.resolve = function (value) {
    return new MyPromise((resolve, reject) => resolve(value))
}
MyPromise.reject = function (reason) {
    return new MyPromise((resolve, reject) => reject(reason))
}

MyPromise.all = function (promiseList = []) {
    const p1 = new MyPromise((resolve, reject) => {
        const result = [] // 存储 promiseList 所有的结果
        const length = promiseList.length
        let resolvedCount = 0

        promiseList.forEach(p => {
            p.then(data => {
                result.push(data)

                // resolvedCount 必须在 then 里面做 ++
                // 不能用 index
                resolvedCount++
                if (resolvedCount === length) {
                    // 已经遍历到了最后一个 promise
                    resolve(result)
                }
            }).catch(err => {
                reject(err)
            })
        })
    })
    return p1
}

MyPromise.race = function (promiseList = []) {
    let resolved = false // 标记
    const p1 = new Promise((resolve, reject) => {
        promiseList.forEach(p => {
            p.then(data => {
                if (!resolved) {
                    resolve(data)
                    resolved = true
                }
            }).catch((err) => {
                reject(err)
            })
        })
    })
    return p1
}
```

