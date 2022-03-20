---
title: 前端面试HTML篇
date: 2021-10-19 19:24:06
tags: 语义化 块级元素 内联元素
---

### 1.如何理解语义化

1）给人看：具有可读性，易懂

2）给搜索引擎看：让搜索引擎读得懂（机器可以识别标签），SEO优化

<!-- more -->

``` html
<div>标题</div>
<div>
    <div>文字内容</div>
    <div>
        <div>列表1</div>
        <div>列表2</div>
    </div>
</div>
```

```html
<h1>标题</h1>
<div>
    <p>文字</p>
    <ul>
        <li>列表1</li>
        <li>列表2</li>
    </ul>
</div>
```



### 2.HTML标签：块级元素和内联元素

1）块级元素：

```html
display：block/table
div h1 h2 table ul ol p 
```

2）内联元素

```html
display：inline/inline-block
button img span input
```

