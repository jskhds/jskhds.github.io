---
title: Flex布局总结
date: 2021-10-19 10:21:14
tags: flex
---

### 1.布局原理

<!-- more -->

- flexible Box 意思是弹性布局

- 通过给父元素添加flex属性，从而控制子盒子的位置和排列方式。任何元素（块级和行内块元素）都可以采用flex布局。采用flex布局以后，float，clear和vertical-align都会失效

#### 1.1container和item

- 采用Flex布局的元素叫做container（容器），它的所有子元素自动称为容器成员，称为flex item （项目）

- item可以纵向排列也可以横行排列

  

### 2.父元素常见属性

| 属性            | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| flex-direction  | 设置主轴方向 row\| row-reverse \| column\|column-reverse     |
| justify-content | 设置主轴上子元素排列方式  flex-start \| flex-end \| center \| space-around \| space-between |
| flex-wrap       | 设置子元素是否换行  nowrap \| weap \| wrap-reverse           |
| align-content   | 设置侧轴子元素的排列方式（多行）                             |
| align-items     | 设置侧轴子元素的排列方式（单行）                             |
| flex-flow       | 复合属性，相当于 flex-direction + flex-wrap                  |

#### 2.1 flex-direction

- 元素是跟着主轴排列的，设置 x 为主轴，那么 y 就是侧轴，反之亦然；
- 属性

| 属性值         | 作用                           |
| -------------- | ------------------------------ |
| row            | 默认主轴为x轴，从左到右        |
| row-reverse    | 从右到左，元素顺序也会跟着翻转 |
| column         | 设置主轴为y轴，从上到下        |
| column-reverse | 从下到上，元素顺序也会跟着翻转 |

#### 2.2 justify-content

- 设置主轴上子元素的排列方式，使用之前要确定好主轴方向
- 属性

| 属性          | 作用                                                         |
| ------------- | ------------------------------------------------------------ |
| flex-start    | 默认值，从左到右/从上到下排列                                |
| flex-end      | 从尾部开始排列                                               |
| center        | 在主轴居中对齐                                               |
| space-around  | 平分剩余空间（eg：主轴是x轴，子元素margin-left和margin-right相同） |
| space-between | 两边贴边，中间元素平分空间（重要）                           |

#### 2.3 flex-wrap 设置子元素是否换行

- 当子元素在父元素中空间不够时，flex布局默认不换行，等比例缩小子元素宽度之后排成一行
- no-wrap 不换行 /   wrap 不够另起一行

#### 2.4  align-items 设置侧轴元素排列方式（单行）

- 设置侧轴元素排列方式 和主轴搭配使用可以实现居正中效果
- 属性  （默认侧轴是 y 轴）

| 属性       | 作用           |
| ---------- | -------------- |
| flex-start | 从上到下       |
| flex-end   | 从下到上       |
| center     | 垂直居中       |
| stretch    | 拉伸（默认值） |

#### 2.5 align-content 设置侧轴元素排列方式（多行）

- 多行：子元素出现要换行的情况 （flex-wrap: wrap;）
- 属性

| 属性          | 作用                       |
| ------------- | -------------------------- |
| flex-start    | 默认从侧轴头部开始排列     |
| flex-end      | 从侧轴尾部开始排列         |
| center        | 在侧轴中间显示             |
| space-around  | 子项在侧轴平分剩余空间     |
| space-between | 两边贴边，中间元素平分空间 |
| stretch       | 设置子项元素平分父元素高度 |

#### 2.6 flex-flow

- eg：flex-wrap: column wrap

### 3.子元素常用属性

- flex items占的份数
- align-self 控制子项自己在侧轴排列的方式
- order 属性定义子项的排列顺序（前后顺序）

#### 3.1 flex 属性

- 分配剩余空间
- flex: number 默认是 0

#### 3.2 align-self 

- 允许单个item与其它 item 有不一样的对齐方式，覆盖align-items属性

#### 3.3 order 

- 定义item的排列顺序
- 数值越小顺序越前