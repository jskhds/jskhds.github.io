---
title: 前端面试CSS 响应式
date: 2021-10-22 19:33:37
tags: rem vw vh响应式
---

### 1.rem是什么

<!-- more -->

##### 1.1 常见单位

- px 绝对长度单位，最常用
- em，相对长度单位，但是相对于父元素，所以不常用
- rem，相对长度单位，相对于根元素，常用

##### 1.2 代码展示

```html
<style>
        html{
        //作为基准的尺寸
            font-size: 100px;
            
        }
        div{
            background-color: #ccc;
            height: 0.3rem;
            font-size: 0.1rem;
            margin-top: 0.1rem;
        }
    </style>
</head>
<body>   
    <div style="width: 0.4rem;">
        div1
    </div> 

    <div style="width: 0.5rem;">
        div2
    </div> 

    <div style="width: 0.6rem;">
        div3
    </div>      
</body>
```



### 2.如何实现响应式

- media-query 根据不同的屏幕宽度设置根元素的font-size
- rem，基于根元素的相关单位

##### 2.1代码示例

```css
@media screen and (max-width: 414px) {
  html {
    font-size: 18px
  }
}

@media screen and (max-width: 375px) {
  html {
    font-size: 16px
  }
}

@media screen and (max-width: 320px) {
  html {
    font-size: 12px
  }
}

```

### 3.vw/vh

##### 3.1 rem的弊端

rem具有“阶梯型”，标准卡得很死，范围不够动态。

##### 3.2 网页视口尺寸（以height为例）

- window.screen.height 整个屏幕高度
- window.innerHeight 网页视口高度，浏览器可以显示内容部分的高度
- document.body.clientHeight  动态的，看内容有多高

##### 3.3 vh和vm

- vh 网页视口高度的1/100
- vm 网页视口宽度的 1/100
- vmax 取两者的最大值；vmin取两者的最小值（一般来说，vh比较大，但是屏幕横过来，vm会比较大）



