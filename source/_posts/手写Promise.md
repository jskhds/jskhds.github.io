---
title: 手写Promise
date: 2021-12-12 10:32:43
tags: 手写Promise
---

手写Promise

<!-- more -->

先给出一个 promise 作为例子

```js
const p1 = new Promise((resolve,reject)=>{
    // resolve(value)
    // reject(reason)
})
```

#### I. 结构

##### 1.myPromise 类

```js
class myPromise  {
    // 1.需要的全局变量
    state = 'pending' // promise 本身有三个状态，所以我们定义一个状态变量 pending fulfilled rejected
    value = undefined // 成功后的值
    reason = undefined // 失败后的原因
    resolveCallbacks = [] // pending 状态下存储成功的回调
    rejectCallbacks = [] // pending 状态下存储失败的回调
    // 2 定义 constructor
    // 2.1 从 p1 看出，constructor 参数是两个函数，这两个函数又分别有自己的参数，一个声明为 value 一个声明为 reason
    constructor(fn){
        const resolveHandler = (value)=>{}  
        const rejectHandler  = (reason)=>{}
    // 2.2 为了函数稳定性 在 try catch 里面执行
        try{  
            fn( resolveHandler,rejectHandler)
           }catch(err)
           { 	
               rejectHandler(err) 
           }   
	}
    // 3 方法 then 和 catch
     then(fn1,fn2){
         
     }
     catch(fn){
         
     }
}
```

##### 2.myPromise 的全局静态API

```js
myPromise.resolve = function(){    } 
myPromise.reject = function(){    } 
myPromise.all = function(){    } 
myPromise.race = function(){    }
```

#### II. 详细代码

##### 1.constructor 部分

考虑以下几点

1）状态只有 pending → fulfilled 和  pending→rejected

2）回调函数数组里面不只一个函数，所以要遍历执行

```js
  constructor(fn){
        const resolveHandler = (value)=>{
            // 执行函数之前，要先判断并且限定状态
            if(this.state === 'pending'){
                this.state = 'fulfilled'
                // 把 value 存储起来
                this.value = value 
                // 在成功的状态下，就执行成功的回调，但是回调函数不止一个，所以要遍历执行 把 value 传进来
                this.resolveCallbacks.forEach(fn => fn(this.value))
            }
        }
        const rejectHandler = (reason)=>{
            if(this.state === 'pending'){
                this.state = 'rejected'
                this.reason = reason
                this.rejectCallbacks.forEach(fn => fn(this.reason))
            }
        }
```

##### 2.方法部分

1） then 里面有两个参数 fn1 fn2 pending状态不执行,把它们存储起来； resolveed状态执行fn1； rejected状态执行 fn2

2）catch 方法相当于 then 的语法糖，因为 then 方法里面本身就可以传进来两个函数，第一个是成功的回调，一个是失败的回调。比如说下面这个例子：then 里面传入两个函数，p2.then() 就执行了后面一个函数。 

```js
const p2 = Promise.reject('my error').catch(err=>{
    throw new Error('catch error')
})
p2.then(()=>{
    console.log('resolved');
},()=>{
    console.log('rejected');
})
```

 3）在 then 方法里，主要部分是判断我们 Promise 的状态，不同的状态做不同的事情

```js
 then(fn1,fn2){
        // 1.因为传入 then 的时候，第一个参数可能是 null 所以需要判断一下类型，以便决定是否执行
        // 1.1如果是函数，就是自己，不是的话，传入什么就是什么 类似于 p.then(v=>v)
        fn1 = typeof fn1 === 'function'? fn1 :(v) => v 
        fn2 = typeof fn1 === 'function'? fn2 :(err) => err 
        // 2.接下来需要判断状态 但不管是什么状态 then 里面不管执行什么代码，返回的都是一个新的 promise  
        // 2.1 pending 存储 fn1 fn2 ，也就是添加到回调列表里面
        if(this.state === 'pending'){
            const p = new myPromise((resolve,reject)=>{
   // 在这里我们只是把函数push进来了，还没有开始遍历执行，当它开始执行的时候，说明状态已经变了，value也就存在
                this.resolveCallbacks.push(()=>{
                    try{
                        const newValue = fn1(this.value)
                        resolve(newValue)
                    }catch(err){
                        reject(err)
                    }
                })
                this.rejectCallbacks.push(()=>{
                    try{
                        const newReason = fn2(this.reason)
                        reject(newReason)
                    }catch(err){
                        reject(err)
                    }

                })
            })
            return p
        }
        // 2.2 fulfilled 执行 fn1
        if(this.state === 'fulfilled'){
            const p1 = new myPromise((resolve,reject)=>{
                try{
                  const newValue =   fn1(this.value) // newValue 是 fn1 用了当前promise的value执行后的值，要返回给新的promise（实现链式调用）
                  resolve(newValue)// 把返回的 newValue 给 resolve 再执行
                }catch(err){
                    reject(err)
                }
            })
            return p1
        }
        // 2.3 rejected 执行 fn2
        if(this.state === 'rejected'){
            const p2 = new myPromise((resolve,reject)=>{
                try{
                    const newReason = fn2(this.reason)
                    reject(newReason) 
                }catch(err){
                    reject(err)
                }
            })
            return p2
        }
    }
```

4）catch 方法 相当于 then 执行第二个函数

```js
catch(fn){
        return this.then(null,fn)
    }
```

##### 3.全局API 部分

1）resolve 和 reject 都比较简单

```js
myPromise.resolve = function(value){
    return new  myPromise((resolve,reject)=>resolve(value))
}
myPromise.reject = function(reason){
    return  new myPromise((resolve,reject)=>reject(reason))
}

```

2.race

```js
// race 只要有一个 fulfilled，就返回 promise
myPromise.race = function(promiseList = []){
        let resolved = false
        const p1 = new myPromise((resolve,reject)=>{
            promiseList.forEach(p=>{
                p.then((data)=>{
                    if(!resolved){
                        resolve(data)
                        resolved = true
                    }
                    
                }).catch((err)=>{
                    reject(err)
                })
            })
        })
        return p1 
    }
```

3)all 

```js
// all 传入 promise 数组，等待所有都 fulfilled 之后，返回新 promise
myPromise.all = function(promiseList = []){
    const p1 = new myPromise((resolve,reject)=>{
        const result = [] // 存储 promiseList 所有的结果
        const length = promiseList.length
        let resolveCount = 0
        // 在 forEach 里面不能用forEach((p,index)=>{})的index来判断是否都执行完，因为 forEach 很快，index瞬间就加满了，但是 then里面的函数不一定执行完了（尤其是有异步函数的情况）
        promiseList.forEach(p=>{
            p.then(data=>{
                result.push(data)
                // resolveCount 只能在then里面加，也就是then执行了才能加
                resolveCount++
                if(resolveCount === length){
                    resolve(result)
                }
            }).catch(err=>{
                reject(err)
            })
        })
    })
    return p1 
}
```

#### III. 全部代码

```js
class myPromise  {
   class myPromise  {
    state = 'pending'  
    value = undefined 
    reason = undefined
    resolveCallbacks = []
    rejectCallbacks = [] 
    constructor(fn){
        const resolveHandler = (value)=>{
            if(this.state === 'pending'){
                this.state = 'fulfilled'
                this.value = value 
                this.resolveCallbacks.forEach(fn => fn(this.value))
            }
        }
        const rejectHandler = (reason)=>{
            if(this.state === 'pending'){
                this.state = 'rejected'
                this.reason = reason
                this.rejectCallbacks.forEach(fn => fn(this.reason))
            }

        }
        try{
        fn( resolveHandler,rejectHandler)
        }catch(err){
            rejectHandler(err)
        }
       
    }
    then(fn1,fn2){
        fn1 = typeof fn1 === 'function'? fn1 :(v) => v 
        fn2 = typeof fn1 === 'function'? fn2 :(err) => err 
        if(this.state === 'pending'){
            const p = new myPromise((resolve,reject)=>{
                this.resolveCallbacks.push(()=>{
                    try{
                        const newValue = fn1(this.value)
                        resolve(newValue)
                    }catch(err){
                        reject(err)
                    }
                })

                this.rejectCallbacks.push(()=>{
                    try{
                        const newReason = fn2(this.reason)
                        reject(newReason)
                    }catch(err){
                        reject(err)
                    }

                })
            })
            return p
        }
        if(this.state === 'fulfilled'){
            const p1 = new myPromise((resolve,reject)=>{
                try{
                  const newValue =   fn1(this.value) 
                  resolve(newValue) 
                }catch(err){
                    reject(err)
                }
            })
            return p1
        }
        if(this.state === 'rejected'){
            const p2 = new myPromise((resolve,reject)=>{
                try{
                    const newReason = fn2(this.reason)
                    reject(newReason) 
                }catch(err){
                    reject(err)
                }
            })
            return p2
        }
    }
    catch(fn){
        return this.then(null,fn)
    }
}
myPromise.resolve = function(value){
    return new  myPromise((resolve,reject)=>resolve(value))
}
myPromise.reject = function(reason){
    return  new myPromise((resolve,reject)=>reject(reason))
}
myPromise.all = function(promiseList = []){
    const p1 = new myPromise((resolve,reject)=>{
        const result = [] 
        const length = promiseList.length
        let resolveCount = 0
        promiseList.forEach(p=>{
            p.then(data=>{
                result.push(data)
                resolveCount++
                if(resolveCount === length){
                    resolve(result)
                }
            }).catch(err=>{
                reject(err)
            })
        })
    })
    return p1 
}
myPromise.race = function(promiseList = []){
        let resolved = false
        const p1 = new myPromise((resolve,reject)=>{
            promiseList.forEach(p=>{
                p.then((data)=>{
                    if(!resolved){
                        resolve(data)
                        resolved = true
                    }
                    
                }).catch((err)=>{
                    reject(err)
                })
            })
        })
        return p1 
    }
```
