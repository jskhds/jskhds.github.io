---
title: React00
date: 2022-01-01 14:17:39
tags: 
---

React概览

<!-- more -->

#### 1.路线

- 基本使用
- 高级特性
- Redux 和 React-router 使用

#### 2.一些面试题

##### 2.1`React` 组件如何通讯

- 父子组件 `props`
- 自定义事件
- `Redux` 和 `Context`

##### 2.2`JSX` 本质是什么 

- `createElement`
- 执行返回 `Vnode`

##### 2.3 Context 是什么，如何应用

- 父组件，向其下所有子孙组件传递信息
- 如一些简单的公共信息：主题色、语言等
- 复杂的公共信息：`Redux`

##### 2.4 `SCU` 的用途

- 性能优化（看实际情况）
- 配合不可变值一起使用，否则会出错

##### 2.5`redux` 单项数据流

比较常考，`React02` 的 `9.6` 图

- `setState` 是同步还是异步

```react
class Root extends Component{
  constructor(props){
    super(props)
    this.state = {count: 0}
  }
  componentDidMount(){
    // 两个异步的合并了
    this.setState({count: this.state.count + 1})
    console.log(this.state.count);  // 0
    this.setState({count: this.state.count + 1})
    console.log(this.state.count); // 0
    // setTimeout 里面的是同步
    setTimeout(() => {
      this.setState({count: this.state.count + 1})
      console.log(this.state.count); // 2
    }, 0);
    setTimeout(() => {
      this.setState({count: this.state.count + 1})
      console.log(this.state.count); //3
    }, 0);   
  }  
  render(){
    return <h1>{this.state.count}</h1>
  }
}
```

##### 2.6 什么是纯函数

- 返回一个新值，没有副作用
- 重点：不可变值
- 比如说： arr1 = arr.slice()

##### 2.7 React 组件生命周期

- 单组件生命周期
- 父子组件生命周期
- 注意 `SCU` 的位置

##### 2.7 React 发起 ajax 应该在哪个生命周期

- 同 `Vue`，在 `DOM` 渲染完成以后
- `componentDidMount`

##### 2.8 渲染列表为何使用 key

- 同 `Vue`。 必须使用 `key`, 并且 `key` 不能是 `Index` 和 `Random`
- `diff算法` 中通过 `tag` 和 `key` 来判断是否是 `sameNode`
- 减少渲染次数，提升渲染性能

##### 2.7 函数组件和 class 组件的区别

- 纯函数，输入 `props`, 输出 `JSX`
- 没有实例，没有生命周期，没有 `state`
- 不能扩展其它方法

##### 2.8 什么是受控组件

- 表单的值，受 `state` 控制
- 需要自行监听 `onChange` 更新 `state`
- 对比非受控组件

##### 2.9 何时使用异步组件

- 同 `Vue`
- 加载大组件
- 路由懒加载

##### 2.10 多个组件有公共逻辑，如何抽离

- 高阶组件 `HOC`
- `Render` `Props`

##### 2.11 redux 如何进行异步请求

- 使用异步 `action`

- `redux-thunk`

```
// redux-thunk 可以返回函数，处理异步请求
```



##### 2.12 react-router 如何配置懒加载

- `lazy`

##### 2.13 PureComponent 有何区别

- 实现了浅比的 `SCU`
- 优化性能
- **结合不可变值使用**

##### 2.14  React 事件和 DOM 事件的区别

- 所有事件挂载在 `root` 上(16挂载在 `document` 上)
- `event` 不是原生的，是 `SyntheticEvent` 合成事件对象
- `dispatchEvent`

##### 2.15 React 性能优化

- 渲染列表用 `key`
- 自定义事件、`DOM` 事件及时销毁
- 合理使用异步组件
- 减少函数 `bind this` 的次数
- 合理使用 `SCU PureComponent memo`
- 合理使用 `Immutable.js`
- `webpack` 层面的优化
- 前端通用的性能优化，如图片懒加载
- 使用 `SSR`

##### 2.16 React 和 Vue 的区别

相同：

- 都支持组件化
- 都是数据驱动视图
- 都是用 `vdom` 来操作 `DOM`

不同：

- `React` 使用 `JSX` 拥抱 `JS`，`Vue` 使用模板拥抱 `html`
- `React` 函数式编程，`Vue` 声明式编程
- `React` 更多需要自己写，`Vue` 有很多内置指令等

