---
title: JS面试篇-Part7
date: 2021-12-10 09:17:09
tags: JS Web API
---

#### 1.与前面知识的联系与区别

- JS基础知识，规定语法（ECMA 262 标准规定）

<!-- more -->

- 变量的类型和计算、原型和原型链、作用域和闭包等
- JS Web API，网页操作的API （W3C 标准规定）
- DOM、BOM、事件绑定、ajax、存储
- 前者是后者的基础，两者结合才能实际应用



#### 2.DOM（Document Object Model）操作知识点

##### 2.1 DOM本质

- HTML是XML的一种，结构和标签都已经规定好了
- DOM 本质是一棵树（和HTML是两个意思，HTML是一个文件，DOM是浏览器根据HTML文件初始化的一棵树）

##### 2.2 DOM节点操作

###### 2.2.1 获取DOM节点

```html
  <!-- CSS -->
  .red {
  	color: red;
}
```



```html
  <!-- HTML结构 -->
  <div id="div1" class="container"></div>
        <p id="p1">一段文字 1</p>
        <p>一段文字 2</p>   
        <p>一段文字 3</p>
    </div>
    <div id="div2" class="container">
        <p>图片</p>
        <img src="../image/xiaoxin.jpg" alt="">         
    </div>
```

```js
// js
// 1. 通过id来获取
const div1 = document.getElementById('div1')
console.log('div1',div1);

// 2. 通过 tagName 获取
const divList = document.getElementsByTagName('div')
console.log('divList length',divList.length);
console.log('divList[0]',divList[0]);

// 3. 通过class获取
const containerList = document.getElementsByClassName('container')
console.log('containerList length',containerList.length);
console.log('containerList[0]',containerList[0]);

// 4. querySelector
const pList = document.querySelectorAll('p')
console.log('pList',pList);


```

###### 2.2.2 修改节点属性  attribute 和 property

- attribute 修改html属性，会改变HTML（DOM树）结构

```js
// attribute 可以作用到DOM结构里面
p1.setAttribute('style','font-size: 50px') // p1 的标签中会添加上修改后的样式
console.log(p1.getAttribute('style'));
```

- property  修改对象属性，不会体现到HTML结构中

```js
const pList = document.querySelectorAll('p')
const p1 = pList[0]
// property 形式 改变页面样式或者渲染结构
p1.style.width = '100px'
console.log(p1.style.width);
p1.className = 'red'
console.log(p1.className);
// 获得nodeName（标签节点的名称）
console.log(p1.nodeName);
// 获得nodeType 一般是 1 不常用
console.log(p1.nodeType);
```

- 二者都有可能引起DOM重新渲染，但是attribute一般来说都会，所以尽量用property操作js

 

##### 2.2 DOM结构操作

- 新增/移动节点

```js
// 1.增加节点
const div1 = document.getElementById('div1')
const newP = document.createElement('p')
newP.innerHTML = 'this is newP'
div1.appendChild(newP)

// 2. 移动节点 获取节点以后 用 appendChild 添加至想要移动到的区域
const p1 = document.getElementById('p1')
const div2 = document.getElementById('div2')
div2.appendChild(p1)
```

- 获取子元素列表，获取父元素

```js
// 3.获取父元素
console.log(p1.parentNode);

// 4. 获取子元素列表 全部获取 不只有 p 标签 还有 text  可以用nodeType 来区分，一般标签为 1，text 为 3
const div1ChildNodes = div1.childNodes
console.log(div1ChildNodes);

// 按照一定规则找出想要的标签
// Array.prototype.slice.call(div1ChildNodes) 把 div1ChildNodes 变成数组
// filter 按照一定条件过滤
const div1ChildNodesP = Array.prototype.slice.call(div1ChildNodes).filter(child=>{
    if(child.nodeType === 1){
        return true
    }
    else{
        return false
    }
})
console.log(div1ChildNodesP );
```



- 删除元素

```js
// 删除一个 p 标签
div1.removeChild(div1ChildNodesP[0])
```



##### 2.3 DOM性能

- DOM 操作非常昂贵，避免频繁的DOM操作
- 对DOM查询做缓存

```js
// 不缓存 DOM 查询结果
for(let i = 0;i<document.getElementsByTagName('p').length;i++){
	// 每次循环都会计算length，频繁进行DOM查询
}

// 缓存 DOM 查询结果
let len = document.getElementsByTagName('p')
for(let i = 0;i<len; i++){
	// 缓存length 只会查询一次DOM
}
```

- 将频繁操作改为一次性操作

eg：给页面插入十个 li 标签，（提前在网页创建一个 id 为 list 的 div）

```js
// 频繁操作的情况 
const list = document.getElementById('list')
for(let i = 0;i<10;i++){
	const li = document.createElement('li')
	li.innerHTML  = `List item ${i}`
	list.appendChild(li)
}
```

```js
const list = document.getElementById('list')
// 创建一个文档片段 此时还没有插入到 DOM 结构中
const frag = document.createElement('fragment')
for(let i = 0;i<10;i++){
	const li = document.createElement('li')
	li.innerHTML  = `List item ${i}`
    // 先把 li 插入到 文档片段中
	frag.appendChild(li)
}
// 最后一次性插入到 DOM 结构中
list.appendChild(frag)
```



##### 2.4 DOM相关问题

- Q: DOM 是哪种数据结构？ 

​		A: 树

-  Q: Property 和 attribute 的区别

    A: property 修改对象属性，不会体现到HTML结构中，attribute 修改 HTML属性，会体现到HTML 结构中

  ​	   二者都有可能引起DOM重渲染，但是 property 主要是在js中修改，性能更好更推荐

-   Q: DOM 性能

​          A: 频繁操作和一次性操作



#### 3.BOM（Browser Object Model）问题

##### 3.1 BOM知识点

-  navigator(识别浏览器的类型) 和 screen

```js
const ua = navigator.userAgent  // navigator.userAgent 获得浏览器的信息，但判断不是很准确，确切的判断浏览器类型等需要具体策略
const isChrome = ua.indexOf('Chrome')
console.log(isChrome)
// screen
console.log(screen.height)
console.log(screen.width)

```

- location (分别拆解 url 各个部分)和 history

```js
console.log(location.href);
console.log(location.protocol);
console.log(location.host);
console.log(location.search);
console.log(location.hash);
console.log(location.pathname);

// history
history.back
history.forward
```



#### 4.事件

##### 4.0 知识点

- 事件绑定
- 事件冒泡
- 事件代理

##### 4.1 事件绑定

```js
// 事件绑定函数(不全面)
const bindEvent = (elem, type, fn)=>{
    elem.addEventListener(type,fn)
}
// eg
const btn1 = document.getElementById('btn1')
bindEvent(btn1,'click',(e)=>{
    console.log(e.target); // 获取点击对象
     e.preventDefault() // 阻止默认行为
     alert('点击成功')
})
```

##### 4.2 事件冒泡

- 浏览器检查实际点击的元素是否在冒泡阶段中注册了一个事件处理程序（同样的激活事件，比如说 click，激活之后要做什么要看具体的回调函数），如果是，则运行它
- 然后它移动到下一个直接的祖先元素，并做同样的事情，然后是下一个，等等，直到它到达`<html>`元素。

```html
<div id="div1">
        <p id="p1">激活</p>
        <p id="p2">取消</p>
        <p id="p3">取消</p>
        <p id="p4">取消</p>
    </div>
    <div id="div2">
        <p id="p5">取消</p>
        <p id="p6">取消</p>
    </div>
```

```js
const body = document.body
const p1 = document.getElementById('p1')
// 1.阻止事件冒泡
bindEvent(p1,'click',e=>{
    e.stopPropagation() // 阻止事件冒泡
    console.log(e.target);
    console.log('激活');
})
// 2.利用事件冒泡机制，多个p标签都实现取消功能，那么给它们的共同祖先绑定可以一次性实现对应功能
bindEvent(body,'click',e=>{
    console.log(e.target);
    console.log('取消');
})

```

##### 4.3 事件代理

- 代码简洁，减少浏览器内存占用
- 不能滥用，有一定复杂度，一般用在瀑布流上

```js
<div id="div3">
        <a href="#">a1</a><br>
        <a href="#">a2</a><br>
        <a href="#">a3</a><br>
        <a href="#">a4</a><br>
        <button>点击加载更多...</button>
    </div>
```

```js
// 事件代理
const div3 = document.getElementById('div3')
bindEvent(div3,'click',e=>{
    e.stopPropagation() // 阻止默认事件：点击 a标签 跳转到链接
    const target = e.target
    if(target.nodeName === 'A'){ // 如果点击了 a标签，进行相应的逻辑，所以需要判断
        alert(target.innerHTML);
    }
})
```



##### 4.4 相关题目

- 编写一个通用的事件监听函数

  - 把 target 改成 this




```js
function bindEvent (elem, type, selector,fn){
    // 只传入三个参数，说明没有选择器
    if(fn == null){
        fn = selector
        selector = null
    }
    elem.addEventListener(type,function(e){
        const target = e.target
        // 代理事件绑定
        if(selector ){
            if(target.matches(selector)){
                fn.call(target,e)
            }         
        }
        // 普通事件绑定
        else{
            fn.call(target,e)
        }
    })
}
```

```js
// eg
const div3 = document.getElementById('div3')
bindEvent(div3,'click','a',function(e){
    e.stopPropagation()
    alert(this.innerHTML)
})
```

- 使用箭头函数（虽然代码量比较少，但是调用的时候不那么优雅）

```js
// 通用的事件绑定函数
function bindEvent (elem, type, selector,fn){
    // 只传入三个参数，说明没有选择器
    if(fn == null){
        fn = selector
        selector = null
    }
    elem.addEventListener(type,e=>{
        fn(e)    
    })
}
```

```js
const div3 = document.getElementById('div3')
bindEvent(div3,'click','a',e=>{
    e.stopPropagation()
    // 用 e.target 表示触发元素
    alert(e.target.innerHTML)
})
```



- 描述事件冒泡的流程
  - 基于DOM树形结构
  - 事件会顺着触发元素向上冒泡
  - 应用场景：事件代理

- 无限下拉的图片列表，如何监听每个图片的点击？
  - 事件代理
  - 用 e.target 获取触发元素
  - 用 matches 来判断是否是触发元素
