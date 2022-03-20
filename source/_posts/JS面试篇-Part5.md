---
title: JS面试篇-Part5
date: 2021-12-04 20:21:45
tags: EventLoop Promise async await for...of
---

#### 1.event loop(事件循环/事件轮询)

<!-- more -->

- JS是单线程运行的

  从前到后逐行执行；某一行报错则停止，不执行后面的代码；先执行同步再执行异步

- 异步要基于回调来实现

- event loop 就是异步回调的实现原理

  演示代码如下

  ```js
  console.log('Hi');
  setTimeout(function cb1() {
      console.log('callback');
  }, 1000);
  console.log('Bye');
  ```

  解释如下：

  1. 同步代码，一行一行放在 Call Stack 里面执行
  2. 遇到异步，先记录下来，等待时机到达（定时，网络请求等）
  3. 时机到了，就移动到 Call Queue 里面
  4. 如果 Call Stack 为空（即同步代码执行完），Event Loop 开始工作
  5. 轮询查找 Callback Queue,如有则移动到 Call Stack 执行
  6. 然后继续轮询查找

  

  #### 2.DOM事件和Event Loop

  - JS是单线程的
  - 异步（setTimeout，ajax）使用回调，基于event loop
  - DOM事件也使用回调，基于event loop （DOM事件不是异步）




```js
 <button id="btn"></button>
    <script>
        let btn = document.querySelector('button')
        console.log('HI');
        btn.addEventListener('click',function(e){
            console.log('button clicked');
        })
        console.log('Bye');
    </script>

```

同步代码立即执行，btn.addEventListener也是立即执行，但是里面的回调函数先进去Call Stack 等待，点击以后进入Web APIs 再进入 Callback Queue 给event loop 执行

#### 3.Promise深入

##### 三种状态

- pending、fulfilled、rejected

```js
// 三种状态
const p1 = new Promise((resolve,reject)=>{

})
console.log('p1',p1);  // p1 Promise { <pending> }

const p2 = new Promise((resolve,reject)=>{
    setTimeout(()=>{
        resolve()
    })
})
console.log('p2',p2); // pending
setTimeout(()=>{
    console.log('p2-setTimeOut',p2);  // resolved （fulfilled）
})

const p3 = new Promise((resolve,reject)=>{
    setTimeout(()=>{
        reject()
    })
})
console.log('p3',p3); // pending
setTimeout(()=>{
    console.log('p3-setTimeout',p3); // rejected
})
```

##### 状态的表现和变化

- pending状态，不会触发 then 和 catch 
- fulfilled 状态，会触发 then 回调函数
- rejected 状态，会触发 catch 回调函数

```js
 // 状态转换 不会触发什么 会触发什么
const p1 =  Promise.resolve(100) // 单独用的时候，promise 不用 new
p1.then((data)=>{
    console.log('data1',data);
}).catch(err=>{
    console.error('err1',err)
})  // resolve 只会执行 then 结果 data1 100

const p2 = Promise.reject('err')
p2.then(data=>{
    console.log('data2',data);
}).catch(err=>{
    console.log('err2',err);
})  // reject 只会执行 catch 结果 err2 err
```



##### then 和 catch 对状态的影响

- then 正常返回 resolved ， 里面有报错则返回 rejected （完整来说eg：then正常返回一个 resolved 状态的 promise）

```js
const p1 = Promise.resolve().then(()=>{
    return 100 // then 正常执行
})
console.log('p1',p1);  // 一开始是pending 执行后是 resolved

const p2 = Promise.resolve().then(()=>{
    throw new Error(['then error'])  // then 里面有报错  返回 rejected
})
console.log('p2',p2);  // 一开始是pending 执行后是 rejected
// 执行
p1.then(data=>{
    console.log(data);  // 因为 p1 = 返回了一个resolved状态的promise 所以可以执行 then
}).catch(err=>{
    console.error('err1',err);  // catch 不会被执行
})

p2.then(data=>{
    console.log(data);
}).catch(err=>{
    console.error('err2',err)  // p2 = 返回了一个rejected 状态的 promise 执行catch 
                                // 打印 err2 Error: then error（p2 throw 出来的错误）
})
```



- catch 正常返回 resolved，里面有报错则返回 rejected

```js
const p3 = Promise.reject('my error').catch(err=>{
    console.error('err',err)
})

console.log('p3',p3);  // reject 是正常执行了的 所以 promise 的状态是 resolved（fulfilled）
p3.then(()=>{
    console.log('100');  // 可以打印出来 100
})
const p4 = Promise.reject('my error').catch(err=>{
    throw new Error('catch error')
})
console.log('p4',p4);  // reject 的 catch 报错，返回一个 rejected 的 promise
```

#### 4.Promise 的状态相关问题

- 第一题

```js
Promise.resolve().then(()=>{
    console.log(1);  // resolve 状态 可以执行then then没有报错，返回 resolve状态的promise
}).catch(()=>{
    console.log(2);
}).then(()=>{
    console.log(3); // 所以上一个then返回的promise执行下一个then的回调，不会执行catch
}) // 最后返回一个 resolve 状态的promise

// 打印 1 3 
```

- 第二题

```js
Promise.resolve().then(()=>{ // resolve 执行 then
    console.log(1);
    throw new Error('erro1') // 报错 返回reject
}).catch(()=>{
    console.log(2); // 执行 catch成功 返回 resolve
}).then(()=>{
    console.log(3); // 所以会执行 then
})

// 打印 1 2 3
```

- 第三题

```js
Promise.resolve().then(()=>{ // resolve 执行 then
    console.log(1);
    throw new Error('error1') // 报错 返回 reject 执行 catch
}).catch(()=>{
    console.log(2); // 成功执行 返回resolve 
}).catch(()=>{
    console.log(3); // 在resolve状态下 不能被执行
})

// 打印 1 2
```



#### 5.async/await

- 异步回调 callback hell
- Promise then catch 链式调用，但也基于回调函数
- async/await 是同步语法，彻底消灭回调函数



```js
function loadImg(src) {
    const res = new Promise((resolve,reject) => {
        const img = document.createElement('img')
        img.onload = () =>{
            resolve(img)
        }
        img.onerror = ()=>{
            const err = new Error(`图片加载失败${src}`)
            reject(err)
        }
        img.src = src
    
})
    return res

}
const url1 = '/image/xiaoxin.jpg'
const url2 = '/image/scene.jpg';      
async function () {
    const  img1 = await loadImg(url1)
    console.log(img1.height,img1.width);
    const  img2 = await loadImg(url2)
    console.log(img2.height,img2.width);
    
}()
```

- async/await 和 Promise 相关的关系

  - 执行 async 函数，返回的是 Promise 对象




```js
async function fn1(){
    return 100 // async 返回的是 Promise 函数
}
console.log('fn1:',fn1()); // Promise { 100 } 
 fn1().then((data)=>{
     console.log(data); // 100
 })
```



- await 相当于 Promise 的 then

```js
// 1.await 相当于 then
async function () { // 立即执行匿名函数
    const p1 = Promise.resolve(300)
    const data1 = await p1 // await 相当于 promise then ，直接拿到 return 的数据
    console.log(data1);  // 300
}()

```

```js
// 2.如果 await 后面跟的不是 promise 的值 相当于封装成 一个 promise
!(async function () {
     
    const data2 = await 400 // await Promise.resolve(400)
    console.log(data2);   // 400
})()
```

```js
// 3. await 后面接一个返回promise对象的函数
!(async function () { 
    const data3 = await fn1()  
    console.log(data3);   // 100
})()
```



- try...catch 可以捕获异常，代替了 Promise 的 catch

```js
//1.try...catch举例
!(async function(){
    const p4 = Promise.reject('err')
    try{
        const res = await p4
        console.log('res',res);
    }catch(ex){ 
        console.error('ex',ex) // ex err
    }
})()
```

```js
// 2.await 的一种冲突，用try...catch来解决
!(async function(){
    const p5 = Promise.reject('err')
    const res = await p5 // 因为 await 相当于 then 所以 rejected状态的promise走不到这一步
    console.log(res);
})
```

#### 6. 异步的本质是回调

- async-await 是语法糖，只是语法层面上像同步，但是很好用
- 代码示例

```js
// async 里面的函数体（除了await）都是立即执行的
async function async1() {
    console.log('async1 start'); //2.
    await async2() // 先执行async函数体内容 然后执行await
    // 因为 async相当于返回 undefined 所以 await后面没有promise
    // 但是 await 后面的语句都可以当做是异步 所以要最后执行
    // 最后一句类似于setTimeout(()=>{ console.log('async1 end');})
    // (当然只是举例，也有可能是promise.then())
    console.log('async1 end');   //5.
}
async function async2() {
    console.log('async2');//3.
    
}
console.log('script start'); //1.
async1()
console.log('script end');//4.

// script start
//async1 start
//async2
//script end
//async1 end
```

```js
async function async1() {
    console.log('async1 start');  // 2
    await async2()  
    // 下面三句都是异步回调
    console.log('async1 end');   // 5
    await async3()  // 6
    // 下面一行是异步回调
    console.log('async1 end2'); // 7
}
async function async2() {
    console.log('async2'); // 3
    
}
async function async3() {
    console.log('async3');
}
console.log('script start');  // 1 
async1()
console.log('script end');  // 4

// script start
// async1 start
// async2
// script end
// async1 end
// async3
// async1 end2
```

#### 7.for...of

for...in 可以把自身和原型链上可枚举的属性都遍历出来，所以一般用来遍历对象的 key

for...of 遍历可迭代对象，得到 value

- for ... in (以及 forEach for) 是常规的同步遍历
- for ... of 常用于 异步的遍历（需要熟练掌握）

```js
function multi(num){
    return new Promise( resolve=>{
        setTimeout(()=>{
            resolve(num*num)
        },1000)
    }         )
}

const nums = [1,2,3]
// 1s 间隔后立即打印出 res 的三个结果 1 4 9
nums.forEach(
    async(i)=>{
        const res = await multi(i)
        console.log(res);
    }
    
)  

// 如果要按照 1s 间隔逐个打印出来 for...of 用在异步当中
!(async function(){
    for(let i of nums){
        const res = await multi(i)
        console.log(res);
    }
})()

```



#### 			

