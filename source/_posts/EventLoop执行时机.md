---
title: EventLoop执行时机
date: 2022-03-16 01:08:06
tags: 面试题
---

```js
async function async1(){
    console.log(1);
    await async2();
    console.log(2);
    
}
async function async2(){
    console.log(3);
}
console.log(4);
setTimeout(()=>{
    console.log(5)
},0);
async1();
new Promise(resolve =>{
    console.log(6);
    resolve();
}).then(()=>{
    console.log(7);
}).then(()=>{
    console.log(8);
})

new Promise(resolve =>{
    console.log(9);
    resolve();
}).then(()=>{
    console.log(10);
}).then(()=>{
    console.log(11);
})
// 4
// 1
// 3
// 6
// 9
// 2
// 7
// 10
// 8
// 11
// 5
```





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



