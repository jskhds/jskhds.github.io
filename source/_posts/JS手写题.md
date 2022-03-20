---
title: JS手写题
date: 2021-12-17 20:39:56
tags: 深拷贝 简单深比较
---

为了方便查找，我把 JS 的一些简单手写题做了一个汇总。 这些题目比较小，但是细节多，需要时时复习

<!-- more -->

##### 1.手写深拷贝

###### 1.1简单版本

在JavaScript中，数组和对象都是引用类型，直接复制的是地址，所以需要深度拷贝以便修改不影响原来的数组或对象。主要运用到递归。深拷贝其实内容很多，这是最简单的版本。

```js
function deepClone(initalObj, finalObj) {    
  var obj = finalObj || {};    
  for (var i in initalObj) {        
    var prop = initalObj[i];        // 避免相互引用对象导致死循环，如initalObj.a = initalObj的情况
    if(prop === obj) {            
      continue;
    }        
    if (typeof prop === 'object') {
      obj[i] = (prop.constructor === Array) ? [] : {};            
      arguments.callee(prop, obj[i]);
    } else {
      obj[i] = prop;
    }
  }    
  return obj;
}
var str = {};
var obj = { a: {a: "hello", b: 21} };
deepClone(obj, str);
console.log(str.a);
```

###### 1.2完善版本

medium 上有一篇文章讲了深拷贝

https://javascript.plainenglish.io/write-a-better-deep-clone-function-in-javascript-d0e798e5f550



##### 2.手写深度比较 （简单版）

网上深度比较有很多的代码，简单的也有，复杂的也有，但是复杂的考虑得太全面了，考虑到我当前水平，还是简洁一点的比较适合我

我对这段代码主要是两个问题：

一是网上很多在判断的地方有一段代码： 判断是否传入同一个对象

```js
if(obj1 === obj2){
        return true
  }
```

 一开始我本地跑了好几次也没有用上过这段，所以就删掉了。

后来我想了想才发现，因为我们在写代码的时候，要从用户的角度出发，也就是我们不知道用户到底会传什么进来，有可能两个参数传递是一个对象，所以这段代码就是为了应对这种情况，不需要再浪费时间遍历递归去比较，还是很有必要的

还有一段是：判断任意一个不为引用类型的时候是否相等，大家都用的 || 

```js
if(!isObject(obj1)&&!isObject(obj2)){       
        return obj1 === obj2
  }
```

这我也觉得很奇怪，如果一个是对象一个不是，那不是全部都是 false 吗？ 后来我换成 && 试了一下，发现也是正确的，只是走的路径不一样。所以这里 || 或者 && 应该都是正确的

```js
// 另写一个函数 判断是否为 不为空的对象或数组
function isObject(obj){
    return typeof obj === 'object' && obj !== null
}
function isEqual(obj1,obj2){
    // 递归终止条件
    if(!isObject(obj1)||!isObject(obj2)){      
        return obj1 === obj2
    }
    if(obj1 === obj2){
        return true
  }
    //  获取所有属性
    const obj1Keys = Object.keys(obj1)
    const obj2Keys = Object.keys(obj2)
    if(obj1Keys.length !== obj2Keys.length){
        return false
    }
    // 递归比较
    for(let key in obj1){
        const res = isEqual(obj1[key],obj2[key])
        if(!res){
            return false
        }
    }
    return true

}

```

##### 3.手写 Promise 异步加载图片

```js
const loadImage= (url)=>{
    const img = document.createElement('img')
    return new Promise((resolve,reject)=>{
        img.onload = ()=>{
            console.log('test')
            resolve(img)
        }
        img.onerror = ()=>{
             const err = new Error('could not load image at' + url)
             reject(err)
        }
        img.src = url
    })
}
// test 
loadImg(url).then(img=>{
    console.log(img.width);
    return img
}).then(img=>{
    console.log(img.height);
}).catch(ex =>{
    console.error(ex)
})
```

##### 4.promise 实现 Ajax （手写 Ajax）

前置知识：需要知道 XMLHttpRequest 的一些基础知识，readyState、status 状态码 以及 post send 等方法

```js
// 手写 ajax
// return 一个 promise 参数 xhr.open  xhr.send  xhr.onreadystatechange
function ajax(url){
    const p = new Promise((resolve,reject)=>{
        const xhr = new XMLHttpRequest()
        xhr.open('GET',url) 
        xhr.onreadystatechange = function(){
            if(xhr.readyState === 4 ){
                if(xhr.status === 200){
                    resolve(
                        JSON.parse(xhr.responseText)
                    )
                }
                else if(xhr.status === 404){
                    reject(new Error('404 not found'))
                }
            }
        }
        xhr.send(null)
    })
    return p
}
const url = '/data/test.json' 
ajax(url).then(res=>{
    console.log(res);
}).catch(err=>{
    console.error(err)
})


```

##### 5. 手写防抖

防抖的功能是用户不停就不执行，等用户停了再操作。用户怎么才算停呢？delay 这么长的时间内都没有触发事件发生就算停了。比如说用户连续输入，需要等用户输入停止一段时间后再打印或者传输数据

```js
function debounce(fn,delay){
    let timer = null
    return function(){
        if(timer){
            clearTimeout(timer)
        }
        timer = setTimeout(()=>{
            // 把 fn 的 this 改成触发元素触发
            fn.apply(this,arguments)
            timer = null
        },delay)
    }
}
inp1.addEventListener('keyup',debounce(function(){
    console.log(inp1.value);
},500))
```



##### 6.手写节流

```js
const div1 = document.getElementById('div1')
function throttle(fn,delay){
    let timer = null;
    return function(){
        if(timer){
            return
        }
        timer = setTimeout(()=>{
            fn.apply(this,arguments)
            timer = null
        },delay)
    }
}
div1.addEventListener('drag',throttle(function(e){
    console.log(e);
    console.log(e.offsetX);
},1000))
```

