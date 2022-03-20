---
title: JS面试篇-Part9
date: 2021-12-11 18:52:57
tags: http
---

#### 0.透视HTTP协议中对 HTTP 的定义：

1.HTTP是一个用在计算机世界里的协议。它使用计算机能够理解的语言确立了计算机之间交流沟通的规范，以及相关的各种控制和错误处理方式

2.HTTP是一个在计算机世界里专门用来在两点之间传输数据的约定和规范

3.HTTP是一个在计算机世界里专门在两点之间传输文字、图片、音频、视频等超文本数据的约定和规范

#### 1.http常见的状态码

<!-- more -->

| 码   | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| 1xx  | 服务器收到请求                                               |
| 2xx  | 请求成功，如 200                                             |
| 3xx  | 重定向，如 301(永久重定向) 302（临时重定向）304(资源未被修改) |
| 4xx  | 客户端错误，如 404(资源未被找到) 403(没有权限)               |
| 5xx  | 服务端错误，如 500(服务器错误) 504(网关错误)                 |

#### 2.http methods Restful  API

- 传统的 methods

  - get 获取数据，post 向服务器发送数据

- 现在的 methods

  - get 获取数据
  - post 新建数据
  - patch/put 更新数据
  - delete 删除数据

- Restful  API

  - 一种新的 API 设计方法（早已推广使用）
  - 传统API设计：把每个 url 当做一个功能
  - Restful  API：把每个 url当做一个唯一的资源

- 如何把 url 设计成一个资源？

  - 不使用 url 参数 
    - 传统 API：/api/list?pageIndex = 2; Restful  API: /api/list/2

  - method 表示操作类型
    - 传统 API： post：/api/create-blog; post: /api/update-blog?id=100; get: /api/get-blog?id=100（url里面就知道要做什么了，也就是功能）
    - Restful  API： post: /api/blog; patch: /apiblog/100; get: /api/blog/100 (只能看出来资源标识也就是100，看请求名，才能知道具体要做什么，增加，更新，获得)

  

#### 3.http headers

- 常见的 Request Headers

| 名称                   | 含义                                |
| ---------------------- | ----------------------------------- |
| Accept                 | 浏览器可接收的数据格式              |
| Accept-Encoding        | 浏览器可接收的压缩算法，如gzip      |
| Accept-Language        | 浏览器接收的语言 如 zh-CN           |
| Connection: keep-alive | 一次TCP链接重复使用                 |
| cookie                 | 同域请求，浏览器都会自带            |
| host                   | 请求的域名                          |
| User-Agent（UA）       | 浏览器信息                          |
| Content-type           | 发送数据的格式，如 application/json |

- 常见的 Response Headers

| 名称              | 含义                                |
| ----------------- | ----------------------------------- |
| Content-type      | 返回数据的格式，如 application/json |
| Content-length    | 返回数据的大小，多少字节            |
| Content- Encoding | 返回数据的压缩算法，如 gzip         |
| Set-Cookie        | 服务端修改 cookie                   |

- 缓存相关的 headers （第四点）
  - cache-control

​	

#### 4.http缓存

- 缓存介绍

  - 把可以重复利用的存储下来，让下次访问页面加载得更快些
  - 网络请求环节加载比较慢，需要提高速度，优化网络请求需要缓存
  - 静态资源可以被缓存，js css img 

- http 缓存策略（强制缓存+协商缓存）

  - 强制缓存

    - 过程：发送请求到服务器，返回带cache-control给浏览器，浏览器下次就向本地缓存请求返回资源（缓存过期则重新向服务器请求）

    - Expires 同在 response headers 中，控制缓存过期，已被 cache-control 替代

    - cache-control（为主）

  | 名称           | 含义                                                   |
| -------------- | ------------------------------------------------------ |
  | max-age        | 缓存最大过期时间                                       |
  | no-cache       | 不用本地（强制）缓存，到服务端请求                     |
  | no-store       | 不用本地缓存，也不用服务端缓存，服务端直接返回资源即可 |
  | private/public | 只能用户终端做缓存/中间代理等也可以做缓存              |
  
  -  协商缓存（对比缓存）
  - 服务端缓存策略（服务端判断资源是否要缓存）
    - 服务端判断客户端资源是否和服务端资源一样
    - 一致则返回 304，否则返回 200 和最新的资源
    - 资源标识
      - 在response headers 中，有两种
      - Last-Modified 资源的最后修改时间
      - Etag 资源的唯一标识（一个字符串，类似于人的指纹）
      - 优先使用Etag，因为Last-Modified 的精度在秒级，精度不高，如果资源被重复生成，而内容不变用Etag比较精确。（因为重复生成时间会变则Last-Modified会变，Etag是基于内容生成的，所以Etag不会变）
  
  ​		

- 刷新方式对缓存的影响

  - 正常操作：地址栏输入 url，跳转链接，前进后退；强制缓存有效，协商缓存有效
  - 手动刷新：F5，右击菜单刷新；强制缓存失效，协商缓存有效
  - 强制刷新：ctrl + f5；强制缓存失效，协商缓存失效

#### 5.https

- http和https

  - http 明文传书，敏感信息容易被中间劫持
  - https：http + 加密，劫持了也无法解密
  - 现代浏览器已经开始强制 https 协议

  

- 加密方式：对称加密，非对称加密

  - 对称加密：服务端和客户端用同一个 key 在传输过程中加密和解密，不安全，因为key也在传输过程中传输
    - 过程
  - 非对称加密：一对 key，pubkey 加密后只能用 key 来解密
    - 过程
  - https 用了这两种方式加密
    - 过程

- https 证书

  - 中间人攻击，把 pubkey 掉包
  - 使用第三方证书
  - 浏览器校验证书
  - 过程
    - 
