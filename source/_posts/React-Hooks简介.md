---
title: React-Hooks简介
date: 2022-02-03 14:56:12
tags: React-Hooks
---

一些常见hook和注意事项

<!-- more -->

#### 0.Hooks

- 消息处理的一种方法，用来监视指定程序
- 函数组件中需要处理副作用，可以用钩子把外部代码“钩”进来
- 常用钩子： `useState useEffect useContext useReducer`
- hooks 一律使用 useXxx 格式

#### 1.State Hook

函数组件：引入 `react`；纯函数；返回 `JSX`；

让函数组件实现 `state` 和 `setState`

- 默认函数组件没有 `state`
- 函数组件是一个纯函数，执行完成即销毁，无法存储 `state`
- `hook` 相当于一个钩子，把 `state` 钩到纯函数中

`useState`

- `useState(0)` 传入初始值，返回数组 `[state, setState]`
- 通过 `state` 获取值
- 通过 `setState(1)` 修改值

代码

```JSX
// State Hook 点击加一
import React ,{useState} from "react";
function ClickCounter(){
    const [count, setCount] = useState(0)
    // count: 0, 修改 count 的方法是 setCount
    // const [name, setName] = useState('Dmoon')
    return <div>
        <p> 共点击{count}次</p>
        {/* setCount(count + 1) 表示对 count 的修改操作 */}
        <button onClick={()=> setCount(count + 1)}>点击</button>
    </div>
}
export default ClickCounter
```



```JSX
// class 组件点击加一
import React from "react";

class ClickCounterClass extends React.Component{
    constructor(props){
        super(props)
        this.state = {
            count: 0
        }       
    }
    render(){ 
        return <div>
            <p>共点击了{this.state.count} 次</p>
            <button onClick={this.ClickHandler}>点击</button>
        </div>
    }
    ClickHandler = () =>{
        this.setState({
            count: this.state.count + 1
        })
    }
}
export default  ClickCounterClass
```

**注意所有的 Hooks 都用 use 开头，包括自定义 Hooks，其它地方不要使用 useXxxx 的写法**

#### 2.Effect Hook

前置知识 `side effect`：纯函数中函数的输入确定那么输出也确定，`side effect` 和纯函数相反，指一个函数处理了与返回值无关的事情

##### 2.1 生命周期

```JSX
import React ,{useState, useEffect} from "react";
function LifeCycles(){
    const [count, setCount] = useState(0)
    const [name, setName] = useState('Dmoon')
    //1.模拟 Class 里的 DidMount 和 DidUpdate，很容易不停发送请求，陷入死循环
    useEffect(()=>{
        console.log('send ajax request');
    })

    // 2.多一个参数 [] 模拟 Class 里的 DidMount  
    useEffect(()=>{
        console.log('send ajax request done');
    },[])

    //3. 数组里加对象模拟 Class 里的 DidUpdate [count,name] 表示 count 和 name 更新了都能触发这个生命周期
    useEffect(()=>{
        console.log('updated');
    },[count,name])

	// 4. willUnMount
    useEffect(()=>{
        let timerId = setInterval(()=>{
            console.log(Date.now());
        },2000)
        // 组件销毁时 模拟 WillUnMount 返回一个函数。
        return ()=>{
            clearInterval(timerId)
        }
    })
    function ClickHandler(){
        setCount(count + 1)
        setName(name + '2022')
    }
    return <div>
        <p> LifeCycles 共点击{count}次</p>
        <button onClick={ClickHandler}>点击</button>
    </div>
}

export default LifeCycles
```

```jsx
// willUnMount周期 父组件的补充
// ...
function App() {
  const [flag, setFlag] = useState(true)
  return (
    <div >
     {flag && <LifeCycles/>}
     {/* 注意不能写成 onClick={setFlag(false)}，必须在匿名函数里面写*/}
     <button onClick={()=>setFlag(false)}>停止打印时间戳</button>
     <button onClick={()=>setFlag(true)}> 开始打印时间戳</button>    
    </div>
  );
}
```

让函数组件模拟**生命周期**，也就是用 `Effect Hook` 把生命周期钩到函数组件中

- 模拟 `componentDidMount` - `useEffect` 依赖 `[ ]`
- 模拟 `componentDidUpdate` - `useEffect` 依赖 `[ ]` 或者 `[a,b]`
- 模拟 `componentWillUnMount` - `useEffect` 返回一个函数

##### 2.2 `Effect Hook` 模仿`componentWillUnMount` 的注意点：

```JSX 
// class 组件实现状态监听
class FriendStatusClass extends React.Component{
    constructor(props) {
        super(props)
        this.state = {
            status: false, // 默认当前不在线
            count: 0
        }
    }
    render(){
        return <div>
           好友 {this.props.friendId} 在线状态:{this.state.status.toString()}  
        </div>
    }
    // 每一个生命周期都需要定义
    componentDidMount(){
        console.log(`开始监听 ${this.props.studentId} 的在线状态 `);
    }
    // 注意更新时前后状态都要注意
    componentDidUpdate(prevProps){
        console.log(`结束监听 ${prevProps.studentId} 的在线状态`);
        console.log(`开始监听 ${this.props.studentId} 的在线状态`);
    }
    componentWillUnmount(){
        console.log(`结束监听 ${this.props.studentId} 的在线状态`);

    }
}
```



```JSX
// 函数组件 + hooks 实现状态监听，代码大大减少
function StudentsStatus(props){
    const [status, setStatus] = useState(false)

    useEffect(()=>{
        // 没有依赖参数，所以既可以监听 didUpdate 也可以监听 didMount
        console.log(`开始监听学生 ${props.studentId}的在线状态`);
         // 【特别注意】
        // 此处并不完全等同于 WillUnMount
        // props 发生变化，即更新，也会执行结束监听
        // 准确的说：返回的函数，会在下一次 effect 执行之前，被执行
        return (()=>{
            console.log(`结束监听 ${props.studentId}的在线状态`);
        })
    })

    return <div>
        学生{props.studentId} 的在线状态: {status.toString()}
        <button onClick={()=> setStatus(!status )}>切换在线状态</button>
    </div>
}
```

##### 2.3 useEffect 使用 async/await 

```JSX
// 比如说从某个 api 获取 data, async 不能直接写在 useEffect 中，而是要另起函数，不然会报错
useEffect(()=>{
     /**
     * 用原生JavaScript 的 fetch 通过 get 请求拿到数据
     * fetch 直接返回的不是数据，而是 promise 所以还需要 then，
     * response 里面有 header status 等 我们拿到 response.json() 也就是主体数据， 返回的是 promise
     * 还需要 then 一下 拿到 data
     */
      const fetchData = async()=>{
        const response = await fetch("https://jsonplaceholder.typicode.com/users")
        const data = await response.json()
        setRobotsGallery(data)
      }
      fetchData()
    },[])
```

##### 2.4 处理异常 try-catch

```JSX
useEffect(()=>{
      const fetchData = async()=>{
        // 获得数据以前 loading 为 true
        setLoading(true)
        try {
          const response = await fetch("https://jsonplaceholder.typicode.com/users")
          const data = await response.json()
          setRobotsGallery(data)
        }catch(e){
          setError(e.message)
        }
        // 获取数据以后 loading 为 false
        setLoading(false)
      }
      fetchData()
},[])
```



#### 3.其它 Hook

##### 3.1 useRef

获得 `DOM` 节点

```JSX
import React ,{useEffect, useRef} from "react"
function UseRefDemo(){
    const btnRef = useRef(null)
    useEffect(()=>{
        console.log(btnRef.current); //  <button>按钮</button>
    })
    return <div>
        <button ref={btnRef}>按钮</button>
    </div>
}
export default UseRefDemo
```



##### 3.2 useContext

```JSX
import React, { useContext } from 'react'

// 主题颜色
const themes = {
    light: {
        foreground: '#000',
        background: '#eee'
    },
    dark: {
        foreground: '#fff',
        background: '#222'
    }
}

// 创建 Context
const ThemeContext = React.createContext(themes.light) // 初始值

function ThemeButton() {
    const theme = useContext(ThemeContext)

    return <button style={{ background: theme.background, color: theme.foreground }}>
        hello world
    </button>
}

function Toolbar() {
    return <div>
        <ThemeButton></ThemeButton>
    </div>
}

function App() {
    return <ThemeContext.Provider value={themes.dark}>
        <Toolbar></Toolbar>
    </ThemeContext.Provider>
}

export default App

```



##### 3.3 useReducer

```JSx
// 两个参数   initialState  和 reducer 
const initialState = {count: 0}
const reducer = (state, action)=>{
    switch(action.type){
        case 'increment':
            return { count: state.count + 1}
        case 'decrement':
            return { count: state.count - 1}
        default:
            return state
    }
}
function ReducerDemo(){
    // useReducer 使用
    const [state, dispatch] = useReducer(reducer, initialState)
    return <div>
        count: {state.count}
        <button onClick={()=>dispatch({type: 'increment'})}>+</button>
        <button onClick={()=> dispatch({type: 'decrement'})}>-</button>
    </div>
}
```

**useReducer 和 redux 的区别**

- `useReducer` 是 `useState` 的代替方案，用于 `state` 复杂变化
- `useReducer` 是单个组件状态管理，组件通讯还需要 `props`
- `redux` 是全局状态管理，多组件共享数据

##### 3.4 useMemo

`hooks` 性能优化，当然不一定必须要进行性能优化，只在有需要的时候考虑

- `React` 默认会更新所有子组件
- `class` 组件使用 `SCU` 和 `PureComponent` 做优化
- `Hooks` 中使用 `useMemo` ，但优化的原理是相同的

##### 3.5 useCallback

和 `useMemo` 差不多，只是 `useMemo`缓存数据，`useCallback` 缓存函数

```JSX
import React ,{useMemo,memo,useCallback} from "react";
import { useState } from "react/cjs/react.development";

// 用 memo 对子组件进行封装
const Child = memo(({info})=>{
    console.log('child render...');
    return <div>
        <p>当前值: {info.name}</p>
    </div>
})

function App(){
    console.log('parent render...');
    const [count, setCount] = useState(0)
    const [name, setName] = useState('Dmoo')
    const [value,setValue] = useState('当前为空')
    // 用 useMemo 对传给子组件的数据进行封装, 依赖里的 item 改变了子组件再会刷新
    const info = useMemo(()=>{
        return {name, age:18}
    },[name])
   // 用 useCallback 封装函数, 父组件层的函数变化不会引起子组件再刷新
    const onChange = useCallback((e)=>{
        setValue(e.target.value)
    },[])
    return <div>
        {count}
        {/* count 改变只有父组件会刷新，子组件不会 */}
        <button onClick={()=> setCount(count + 1)}>点击</button>
        <input value={value} onChange={onChange}></input>
        <p>{value}</p>
        <Child info = {info}/>
    </div>
}

export default App
```

#### 4.自定义 Hook

- 封装通用功能
- 开发和使用第三方 `hooks`
- 自定义 `hook` 带来了无限的扩展性，解耦代码

```JS
import { useState, useEffect } from 'react'
import axios from 'axios'

// 封装 axios 发送网络请求的自定义 Hook
function useAxios(url) {
    const [loading, setLoading] = useState(false)
    const [data, setData] = useState()
    const [error, setError] = useState()

    useEffect(() => {
        // 利用 axios 发送网络请求
        setLoading(true)
        axios.get(url) // 发送一个 get 请求
            .then(res => setData(res))
            .catch(err => setError(err))
            .finally(() => setLoading(false))
    }, [url])

    return [loading, data, error]
}

export default useAxios

// 第三方 Hook
// https://nikgraf.github.io/react-hooks/
// https://github.com/umijs/hooks

```

```JSX
// 使用
import React from 'react'
import useAxions from '...'

function App(){
    const url = 'xxx'
    const [loading, data, error] = useAxions(url)
    if(loading) return <div>loading...</div>
    return error
        ?<div>{JSON.stringfy(error)}</div>
        :<div>{JSON.stringfy(data)}</div>
}
```



#### 5.组件逻辑复用

- `HOC`

  组件层级嵌套过多，不易渲染，不易调试

- `Render Prop`

  学习成本比较高

- `hooks` 实现

  1. 完全符合 `hooks` 原有原则
  2. 变量作用域明确
  3. 不会产生组件嵌套
  4. 代码示例 自定义组件获得鼠标位置




```jsx
import { useState, useEffect } from 'react'
function useMousePosition() {
    const [x, setX] = useState(0)
    const [y, setY] = useState(0)

    useEffect(() => {
        function mouseMoveHandler(event) {
            setX(event.clientX)
            setY(event.clientY)
        }

        // 绑定事件
        document.addEventListener('mousemove', mouseMoveHandler)
        // 解绑事件
        return () => document.removeEventListener('mousemove', mouseMoveHandler)
    }, [])

    return [x, y]
}

export default useMousePosition


// 使用
function App(){
    const [x,y] = useMousePosition()
    return <div>
        <div>鼠标位置 {x} {y}</div>
    </div>
}

```



#### 6.规范和注意事项

1. 命名用 use 开头

2. 使用规范

   - 只能用于 `React` 函数组件和自定义 `hook` 中
   - 只能用于顶层代码，不能在循环、判读中使用 `Hooks`

3. `Hooks` 调用顺序

   - 无论是 `render` 还是 `re-render`， `Hooks` 调用顺序都必须一致
   - 如果 `Hooks` 出现在循环、判断里则无法保证顺序一致
   - `Hooks` 严重依赖于调用顺序

4. 注意事项

   - `useState` 初始化值，只有第一次有效



  

```JSX
// 子组件
function Child({ userInfo }) {
    // render: 初始化 state
    // re-render: 只恢复初始化的 state 值，不会再重新设置新的值
    //            只能用 setName 修改
    // 也就是说只能拿到第一次 userInfo.name 的值，即使父组件 userInfo.name 改变了也不影响子组件
    const [ name, setName ] = useState(userInfo.name)

    return <div>
        <p>Child, props name: {userInfo.name}</p>
        <p>Child, state name: {name}</p>
    </div>
}


function App() {
    const [name, setName] = useState('Dmoon')
    const userInfo = { name }

    return <div>
        <div>
            Parent &nbsp;
            <button onClick={() => setName('慕课网')}>setName</button>
        </div>
        <Child userInfo={userInfo}/>
    </div>
}

```



- `useEffect` 内部不能修改  `state`

  有依赖的时候可以拿到外面的值，如果没有依赖的话，相当于一直在闭包里面执行，就一直拿不到外面的值。可以用 `useRef` 解决



```JSX
import React, { useState, useRef, useEffect } from 'react'

function UseEffectChangeState() {
    const [count, setCount] = useState(0)

    // 模拟 DidMount
    const countRef = useRef(0)
    useEffect(() => {
        console.log('useEffect...', count)

        // 定时任务
        const timer = setInterval(() => {
            console.log('setInterval...', countRef.current)
            // setCount(count + 1)
            setCount(++countRef.current)
        }, 1000)

        // 清除定时任务
        return () => clearTimeout(timer)
    }, []) // 依赖为 []

    // 依赖为 [] 时： re-render 不会重新执行 effect 函数
    // 没有依赖：re-render 会重新执行 effect 函数

    return <div>count: {count}</div>
}

export default UseEffectChangeState

```



- `useEffect` 可能出现死循环

  `React` 使用`Object.is()` 来判断是否相等，所以 如果依赖是值类型内容相等不会更新，但如果依赖是对象数组等引用类型，即使内容一样 `Object.is()` 判断出来都是 false。那么 `useEffect`  就会更新，很容易出现死循环。

#### 7.一些题目

##### 7.1 为什么会有 react hooks ，它解决了什么问题

1. 完善函数组件的能力，函数更适合 `React` 组件
2. 组件逻辑复用， `Hooks` 表现更好
3. `class` 组件比较费解，不易拆解测试
   - `class` 组件中，相同的逻辑散落在各处
   - `DidMount` 和 `DidUpdate` 中获取数据
   - `DidMount` 绑定事件， `willUnMount` 解绑事件
   - 使用 `Hooks`，相同逻辑可以分割到一个个 `useEffect` 中

##### 7.2 React Hooks 如何模拟组件生命周期

看 2

##### 7.3 如何自定义 Hook？

##### 7.4 React Hooks 性能优化

##### 7.6 使用 React Hooks 遇到什么坑？

##### 7.7 Hooks 相比 HOC 和 Render Prop 有哪些优点？（important）

