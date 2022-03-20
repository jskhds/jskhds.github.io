---
title: JS面试篇-Part4
date: 2021-12-04 17:28:51
tags: 同步 异步
---

#### 1.单线程和异步

<!-- more -->

##### 1.1 单线程

- JS是单线程语言，同一时间只能做一件事，会阻塞代码执行
- 因为JS可以修改DOM结构，所以JS和DOM渲染共用一个线程
- 浏览器和nodejs已经支持JS启动进程，如Web Worker
- 补充：一个进程可以开启多个线程，但JS是单线程语言，所以如果要同时多个任务，只能开启多个进程

##### 1.2 为什么要异步

- 遇到等待（网络请求，定时任务） 不能卡住
- 基于callback来实现，不会阻塞代码执行

#### 2.应用场景

- 定时任务 如 setTImeout

```js
  // setTimeout 
  console.log(100);
  setTimeout(function() {
 	console.log(200);
  },1000)
  console.log(300);
```




```js
  // setInterval 循环执行 每隔 interval 执行一次
  console.log(100);
  setInterval(() => {
      console.log(200);
  }, 1000);
  console.log(300);
```

  

- 网络请求 如 ajax图片加载



```js
// ajax 图片加载
console.log("start");
let img = document.createElement('img')
img.onload = function (){
    console.log('loaded');
   
}
img.src = '/xxx.png'
console.log("end");
```



#### 3.callback hell 和 Promise

- callback hell



```js
$.get(url1,(data1)=>{
    console.log(data1);
    $.get(url2,(data2)=>{
        console.log(data2);
        $.get(url3,(data3)=>{
            console.log(data3);
        })
    })
})
```

- Promise



```js
function getData(url ) {
    return new Promise((resolve,reject)=>{
        $.ajax({
            url,
            success(data){
                resolve(data)
            },
            error(err){
                reject(err)
            }
        })
    })}
    
    //使用
    
const url1 = '/data1.json'
const url2 = '/data2.json'
const url3 = '/data3.json'
getData(url1).then(data1=>{
    console.log(data1);
    return getData(url2);
}).then(data2=>{
    console.log(data2);
    return getData(url3)
}).then(data3=>{
    console.log(data3);
}).catch(err => console.error(err))
```



#### 问题

- 同步和异步的区别是什么

​			 如上

- 手写promise加载一张图片			

```js
const url = ' '
        // new Promise格式分析
         function loadImg(src) {
    	   const res = new Promise((resolve,reject) => {
            const img = document.createElement('img')
            // img 加载完成以后执行 resolve 函数
            img.onload = () =>{
                resolve(img)
            }
            // img 加载失败执行 reject 函数
            img.onerror = ()=>{
                const err = new Error(`图片加载失败${src}`)
                reject(err)
            }
            img.src = src  
    	})
    	return res
		}

loadImg(url).then(img=>{
    console.log(img.width);
    // return 的 数据给下一个 .then 用
    return img
}).then(img=>{
    console.log(img.height);
}).catch(ex =>{
    console.error(ex)
})
```

- 加载多张图片

```js
// 加上面的代码 
const url2 = '/image/scene.jpg'
loadImg(url1).then(img1=>{
     console.log(img1.width);
     return img1 // 返回普通对象
 }).then(img1=>{
     console.log(img1.height);
     return loadImg(url2) // 返回 promise 实例
 }).then(img2=>{
     console.log(img2.width);
     return img2
 }).then( img2=>{
     console.log(img2.height);
 }).catch(err=>{
     console.error(err)
 })
```

- 前端使用异步的应用场景

  ​    各种等待的场景

- setTimeout 笔试题

  ```js
  console.log(1);
  setTimeout(() => {
      console.log(2);
  }, 1000);
  setTimeout(() => {
      console.log(3);
  }, 0);
  console.log(4);
  
  // 1 4 3 2
  ```

  

