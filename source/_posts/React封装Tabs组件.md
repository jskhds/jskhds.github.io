---
title: React封装Tabs组件
date: 2022-03-20 13:01:58
tags: 组件封装
---

简单Tabs组件的实现

<!--more-->

#### 1.直接思路

把 tabs 的标签和页面写在一起，用 value 的值来控制显示，这样写比较简单，但是复用性不强，而且如果页面一多手动写起来就会很麻烦。

`Tab`

```React
import React, { useState }from 'react'
import styles from './Tap.module.css'; 
function Tap(){   
    const [value, setValue] = useState(0);
    return <>
    <div className={styles["container"]}>
        <div className={styles.tabList}>
            <div className={styles.item} onClick={()=> setValue(0)}>Tab0</div>
            <div className={styles.item} onClick={()=> setValue(1)}>Tab1</div>
            <div className={styles.item} onClick={()=> setValue(2)}>Tab2</div>
        </div>
        <div className={styles.tabsPanel}>
            {value === 0 && <div className={styles["tabs-panel-item"]}>显示0</div>}
            {value === 1 && <div className={styles["tabs-panel-item"]}>显示1</div>}
            {value === 2 && <div className={styles["tabs-panel-item"]}>显示2</div>}
        </div>
        </div>
    </>
}
export  default Tap;
```

`Tap.module.css`

```css
.tabList{
    width: 400px;
    height: 60px;
    display: flex;
    flex-direction: row;
    align-items:center;
}

.item{
    float: left;
    flex: 1;
    height: 60px;
    line-height: 40px;
    text-align: center;
    border: 1px #ccc solid;
    background-color: skyblue;
    margin-left: 2px;
    
}
.tabsPanel{
    overflow: hidden;
    line-height: 100px;
    height: 100px;
    text-align: center;
}

.tabs-panel-item{
    width: 400px;
    height: 100px;
    background-color: pink;
}
```

#### 2.封装

我封装的思路主要是这样的：

1.从需求上来讲，使用者肯定需要自定义 tabs 标签的数量，所以暴露出的 props 就是让用户传入一个对象数组。对象数组的格式是这样的：

```json
[{
	id:"",
	titile:''
},
defaultID:'' /*可选*/
]
```

2.从封装角度来讲，我要用一个循环来遍历渲染 title，这一步对用户传进来的数组操作即可； 其次要对应 title 和用户需要显示的页面，这里我用的是 react 提供的 props.children，用 index 去对应。(我感觉react 里的 props.children 和 vue 里面的插槽差不多，都是占位符。当然肯定还有些其它功能，不过暂时用不上。）

`TabOptimization`

```React
import  React, {useState} from "react";
import styles from "./TabOptimization.module.css"
function Tabs(props){
    const { list } = props;
    // 当前选择的默认 id 如果用户有传的话就用用户的，没有就默认头一个
    const defaultID = props.defaultID? props.defaultID:list[0].id;
    // 点击之后设置 id 切换
    const [ selected, setSelected] = useState(defaultID);
    // 点击之后拿到 index，用于切换对应页面
    const [index, setIndex] = useState(0);
    const reSet = (id,index) =>{
        setSelected(id);
        setIndex(index);      
    }
    return <>
        <div className={styles["Tabs"]}>
            <div className={styles["Tabs-title"]}>
            {
                list.map((item,index)=>{         
                    return <div key={item.id}
                    onClick={()=>reSet(item.id,index)}
                    className= {item.id === selected? styles["item-active" ]: styles["item-basic title"]}
                    >   
                    {item.name}   
                    </div>
                })
            }
            </div>
            <div className={styles["Tabs-item"]}>{props.children[index]}</div>
        </div>
       
       
    </>
}

export default Tabs;
```

`module.css`

```css
.Tabs{
  width: 200px;
  height: 50px;
  background-color: #fff;
  
}

.Tabs-title{
  display: flex;
  align-items: center;
}
.Tabs-title>div{
  flex: 1;
  text-align: center;
  line-height: 20px;
}
.item-active{
  border-bottom: 1px solid red;
  
}

.item-basic {
  border-bottom: 1px solid #ccc;
  
}
.Tabs-item{
  margin-top: 10px;
  
}
```



使用

```react
function App(){
  const list = [
    {
      id:"1",
      name: "城市1"
    },
    {
      id:"2",
      name: "城市2"
    },
    {
      id:"3",
      name: "城市3"
    },


  ];
  return <>
  <TabOptimization
  list={list}
  >
    <div> className1
      <div>
        test
      </div>
    </div>
    <div> className2</div>
    <div> className3</div>
  </TabOptimization>

  </>
```

效果

![](..\images\tabs封装.png)

#### 3.不足

和 ant-design 这些比起来，我封装的这个组件只能是实现了最基础的功能，而且用户还需要自己传数组，需要优化。再一个其实应该可以暴露一个 callback 给用户，但是我还没想到要怎么做，之后想到了再继续看看吧。
