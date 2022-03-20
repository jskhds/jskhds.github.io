---
title: JS面试篇-Part8
date: 2021-12-11 10:39:39
tags: ajax 跨域 同源策略
---

#### 1. ajax

<!-- more -->

##### 1.1 知识点

- XMLHttpRequest

  - 1.前置知识

    - xhr.readyState 的情况

    | 状态码 | 含义                                                    |
    | ------ | ------------------------------------------------------- |
    | 0      | （未初始化），还没有调用send() 方法                     |
    | 1      | （载入）已调用send()方法，正在发送请求                  |
    | 2      | （载入完成）send() 方法已经执行完成，接收到全部响应内容 |
    | 3      | （交互）正在解析响应内容                                |
    | 4      | （完成）响应内容解析完成，可以在客户端调用              |

    - xhr.status

    | 状态码 | 含义                                                         |
    | ------ | ------------------------------------------------------------ |
    | 2xx    | 请求处理成功（200）                                          |
    | 3xx    | 重定向，浏览器直接跳转，301永久，302暂时跳一次，304资源未改变 |
    | 4xx    | 客户端请求错误 404请求地址错误，403客户端没有权限            |
    | 5xx    | 服务器端错误                                                 |

    

  - 2.请求 get，post，send

```js
// get 请求
const xhr = new XMLHttpRequest()
// 方法，地址，true表示异步请求（false表示同步请求）
xhr.open('GET','/data/test.json',true)
xhr.onreadystatechange = function(){
    if(xhr.readyState === 4){
        if(xhr.status === 200){
            alert(xhr.responseText)
        }else{
            console.log('错误');
        }
    }
}
// 不需要send
xhr.send(null)
```



```js
// post 请求
xhr.open('POST','/login',true)
xhr.onreadystatechange = function(){
// 记得要判断状态
    if(xhr.readyState === 4){
        if(xhr.status === 200){
            console.log(
                JSON.parse(xhr.responseText)
            );   
        }else{
            console.log('错误');
        }
    }
}
const postData = {
    userName: "xxx",
    password: "xxx"
}
// 记得send的是字符串
xhr.send(JSON.stringify(postData))
```

-  同源策略，跨域

  - 同源策略：

    - ajax 请求时，浏览器要求当前网页和server必须同源（安全），针对的是浏览器环境
    - 搜索引擎，爬虫等是从服务端发送的，一般来说不被同源策略限制
    - 同源：协议、域名、端口，三者必须一致
    - eg 前端：http://a.com:8080/; server: https://b.com/api/xxx  (协议域名端口都不同)

    - 加载图片 css js 可无视同源策略 

      | 标签         | 作用                           |
      | ------------ | ------------------------------ |
      | img          | 统计打点，可使用第三方统计服务 |
      | link，script | 使用CDN（一般都是外域）        |
      | script       | 实现JSONP                      |

  - 跨域

    - 所有的跨域都必须经过 server 端允许和配合
    - 未经 server 端允许就实现跨域，说明浏览器有漏洞

-  跨域的实现

  - JSONP

    - 服务器可以任意动态拼接数据返回，只要符合 html /js 格式要求，并不一定是一个 html /js 文件

```js
// 原理：利用 script 实现跨域，服务端动态拼接数据返回
<p>一段文字</p>
    <script>
        // callback 要挂在 window 上才可以全局调用
        window.callback = function(data){
            console.log(data);
        }
    </script>
    // 下面的端口和我们本身的html端口不一样，所以用 script 访问
    <script src="http://localhost:8002/jsonp.js"></script>
```

- CORS 服务器设置 http header

##### 1.2 问题

- 手写一个ajax

```js
// 手写 ajax
function ajax(url){
    const p = new Promise((resolve,reject)=>{
        const xhr = new XMLHttpRequest()
        //xhr.open()的第三个参数为布尔值，可选，默认为true表示是否异步执行操作，默认为true。如果值为false，send()方法直到收到答复前不会返回。如果true，已完成事务的通知可供事件监听器使用。
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

const url = '/data/test.json' // 正确的 url 和错误的 url
ajax(url).then(res=>{
    console.log(res);
}).catch(err=>{
    console.error(err)
})


```



- 跨域的实现方式
  - JSONP（理解原理）
  - CORS 纯服务端

##### 1.3 常用插件

- jquery
- fetch
- axios

#### 2.存储

##### 2.1 知识点

- cookie
  - 实际上用于 浏览器 和 server 之间通讯，但可以被借用到存储中
  - 前端可以用 document.cookie = '' 添加， 修改，获得，只要不删除存储的内容就一直在
  - 缺点：最大存 4kb；http请求时需要发送到服务端，增加请求数据量；只能用 document.cookie 来修改，不好用
- localStorge 和 sessionStorage
  - HTML 专门为存储设计，最大可存 5M
  - API 简易可用 setItem getItem
  - 不会随着 http 发送出去
  - 区别：localStorage 数据会永久存储，除非代码手动删除；sessionStorage 数据存在于当前会话，浏览器关闭就清空
  - localStorage 使用的比较多

#### 2.2 问题 

- cookie localStorage sessionStorage 的区别
  - 存储大小
  - API易用性
  - 是否随http发送

