---
title: 前端面试CSS 图文样式
date: 2021-10-21 19:33:00
tags: line-height的继承问题
---

### 1.line-height的继承问题

<!-- more -->

```html
body{
	font-size: 20px;
	line-height: 200%;
}
p{
	font-size:16px;
}
<body>
	<p>AAA</p>
</body>
```

Q: p 标签的行高是多少？

A：40px 

分析：

- 具体数值，则直接继承
- 写比例，直接继承  本身*比例
- 写百分比，先算完再继承（考点）
