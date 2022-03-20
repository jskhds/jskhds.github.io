---
title: 前端面试真题-part4
date: 2021-12-15 20:37:06
tags: 
---

主要内容：捕获异常，JSON ，获取 URL 参数，手写深拷贝

<!-- more -->



#### 1-part1

##### 1）如何捕获 JS 程序中的异常

solution1 手动捕获(比较长的一串，所以用在高风险区域)

```js
try{
    // todo
}catch(ex){
    console.error(ex)
}finally{
    // todo
}
```

solution 2 自动捕获

```js
window.onerror = function(message,source,linNum,colNum,error){
    // 第一 对于跨域的 js ，比如 CDN 的，不会有详细的报错信息
    // 第二，对于压缩的 js ，还要配合 sourceMap 反差到未压缩代码的行、列
}
```

##### 2）什么是 JSON

- JSON 是一种数据格式，本质是一段字符串
- JSON 格式和 js 对象结构一致，对 JS 语言更友好
- window.JSON 是一个全局对象：JSON.stringify     JSON.parse

```js
{
    "name": "zhangsan",
    "info": {
        "single": true,
        "age": 30
        "city": "Beijing"
    },
    "like": ["music",'basketball']
}
```

##### 3）获取当前页面 url 参数

- 传统方式: location.search

假设 HTML 文件的参数列表为 ?a=10&b=20&c=30  （自己定义就是在 地址栏html 后面加上这一串）

```js
function query(name){
    const search = location.search.substring(1) // 去掉 search 前面的问号
    // search = 'a=10&b=20&c=30'
    const reg = new RegExp(`(^|&)${name}=([^&]*)(&|$)`,'i')
    const res = search.match(reg)
    if(res === null){
        return null
    }
    return res[2]
}
const res = query('a')
console.log(res); //10
const res1 = query('d')
console.log(res1); // null
```

- 新 API: URLSearchParams (兼容问题)

```js
function query(name){
    const search = location.search
    const p = new URLSearchParams(search)
    return p.get(name)
}
res = query('a')
console.log(res);
```



#### 2-part2

##### 1）将 url 解析为 JS 对象 

其实和获取当前页面 url  参数是一个意思，只不过不用正则表达式，用一个对象来接收

```js
function queryToObj(){
    const res = {}
    const search = location.search.substring(1)
    // 'a=10&b=20&c=30'
    search.split('&').forEach(ele=>{
        const arr = ele.split('=')
        const key = arr[0]
        const val = arr[1]
        res[key] = val        
    })
    return res
}

console.log(queryToObj());
```

solution 2

```js
function queryToObj(){
     const res = {}
     const pList = new URLSearchParams(location.search)
     pList.forEach((key,val)=>{
         res[key] = val
     })
     return res
}
```

##### 2）手写 flatern 考虑多层级

```js
function flat(arr){
    let isDeep = arr.some(item=> item instanceof Array)
    if(!isDeep){
        return arr
    }
    // Array.prototype.concat.apply([],arr) 只能拍平两层 所以要递归
    let res = Array.prototype.concat.apply([],arr)
    return flat(res)
}
```

##### 3）数组去重

solution 1  // 用 indexOf 判断 res 里面有没有这个值

```js
function unique(arr){
    const res = []
    arr.forEach(item => {      
        if(res.indexOf(item)<0){
            res.push(item)
        }
    });
    return res
}
```

solution 2  使用 set（无序结构，不允许由于重复元素）

```js
function unique(arr){
     const set = new Set(arr)
     return [...set]
}

```

#### 3-part3

##### 1）手写深拷贝 必会！！！

注意 Object.assign 不是深拷贝，只是拷贝了第一层

```js
function deepClone(obj){
    if(typeof(obj)!=='object' || typeof(obj) == null){
        return obj;
    }
    let res = obj instanceof Array?[]:{};
    for(let key in obj){
        // 确保key不是原型链上的
        if(obj.hasOwnProperty(key)){
            // 递归（重点） key要一层一层遍历，比如说 obj{ address:{city: 'Beijing'}}
            res[key] = deepClone(obj[key])
        }
    }   
    return res;
}
```

##### 2）介绍 RAF requestAnimationFrame

- 动画流畅，更新频率要达到 60帧/s ，即 16.67ms 更新一次视图
- setTimeout 需要手动控制频率，而 RAF 浏览器会自动控制
- 后台标签隐藏在 iframe 中，RAF 会暂停，而 setTimeout 依然执行

```js
// html 里有一个 div 标签
const $div1 = $('#div1')
const max = 640
let curWidth = 100
function animate(){
    curWidth = curWidth + 3
    $div1.css('width',curWidth)
    if(curWidth<max){
    // 如果是 setTimeout 的写法 setTimeOut(animate,16.7) 16.7这个数字要自己计算
        window.requestAnimationFrame(animate)
    }
}
animate()

```

##### 3）性能优化从哪几个方面考虑

- 原则：多使用内存，缓存，减少计算，减少网络请求
- 方向：加载页面，页面渲染，页面操作流畅度















