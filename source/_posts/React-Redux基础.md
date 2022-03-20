---
title: React-Redux基础
date: 2022-01-30 22:36:35
tags: redux react-redux
---

React 小书学习笔记

<!--more-->

#### 1. Redux

Redux 统一保存数据，在隔离了数据与UI的同时，负责处理数据的绑定。它把组件数据(state)剥离出来，将这些数据统一存放在 store 中，组件订阅 store 获得更新后的数据，或者说 store 更新数据之后，会通知所有订阅了它的组件进行更新。

`Redux` 是一种设计模式，不是 `React` 专属，旨在管理共享数据

##### 1.1 dispatch

`state` 里面的某些数据需要共享，但不能随意修改，所以就需要一个专门的函数来管理修改数据的操作，限制我们修改数据的自由度。

##### 1.2 store

- `state` 和  `dispacth` 的集合

- 订阅者模式监听数据变化

在 `createStore` 中，我们的代码只能修改数据，之后还需要我们手动渲染页面，很不方便。所以我们需要在 `createStore `函数中添加订阅者模式，自动渲染页面。

具体来讲，我们只需要添加一个 `listeners` 数组，和 `subscribe` 函数就行。我们执行一次 `createStore` 里的 `subscribe` 函数 ，让 `listeners` 数组添加一个 `listener`函数，然后`dispatch`时遍历 `listeners` 数组，依次执行 `listener` 就能自动渲染页面。

```javascript
function createStore (state, stateChanger) {
  // listeners 数组，存储 listener 渲染页面的函数
  const listeners = []
  // subscribe 接收 listener 参数，push 进 listeners 数组中
  const subscribe = (listener) => listeners.push(listener)
  const getState = () => state
  const dispatch = (action) => {
    stateChanger(state, action)
    // dispatch 时遍历执行 listener
    listeners.forEach((listener) => listener())
  }
  return { getState, dispatch, subscribe }
}
```

​      

```js
const store = createStore(appState, stateChanger)
// 执行 subscribe 函数，添加需要的渲染函数
store.subscribe( () => renderApp(store.getState()) )
renderApp(store.getState()) // 首次渲染页面
store.dispatch({ type: 'UPDATE_TITLE_TEXT', text: '《React.js 小书》' }) // 修改标题文本
store.dispatch({ type: 'UPDATE_TITLE_COLOR', color: 'blue' }) // 修改标题颜色
// ...后面不管如何 store.dispatch，都不需要重新调用 renderApp
```

##### 1.3 纯函数

相同的输入，总是会的到相同的输出，并且在执行过程中没有任何副作用。

副作用指的是函数在执行过程中产生了**外部可观察变化**：

1. 发起HTTP请求
2. 操作DOM
3. 修改外部数据
4. console.log()打印数据
5. 调用Date.now()或者Math.random()

##### 1.4 性能优化

首先，先了解 ES6 的对象浅复制语法

```js
const obj1 = {
    address:{
        city: 'beijing',
        detail: 'chaoyang'
    },
    code: '00000'
    
}
// 已修改数据：把需要修改的数据都浅复制一遍再修改，这样对象的内容不一样，指向也不一样，不再相等
// 未修改数据：内容一样，指向一样，仍然相等
const obj2 = {
    ...obj1,
    address:{
        ...obj1.address,
        city: 'shanghai'
    }
}
console.log(obj1.address.detail === obj2.address.detail);  // true
console.log(obj1.address.city === obj2.address.city); // false
console.log(obj1); //{ address: { city: 'beijing', detail: 'chaoyang' }, code: '00000' }
console.log(obj2); // { address: { city: 'shanghai', detail: 'chaoyang' }, code: '00000' }
```

我们之前的函数，其实很多数据都是没有修改的，完全不需要重新渲染，所以实际上，我们还需要定义 `oldAppState` 和 `newAppState` , 用于比较到底哪些数据进行了修改，哪些没有修改。重新渲染被修改了的数据，未被修改的数据就不用重新渲染。 

```js
function createStore (state, stateChanger) {
  const listeners = []
  const subscribe = (listener) => listeners.push(listener)
  const getState = () => state
  const dispatch = (action) => {
    state = stateChanger(state, action) // 覆盖原对象
    listeners.forEach((listener) => listener())
  }
  return { getState, dispatch, subscribe }
}

function renderApp (newAppState, oldAppState = {}) { // 防止 oldAppState 没有传入，所以加了默认参数 oldAppState = {}
  if (newAppState === oldAppState) return // 数据没有变化就不渲染了
  console.log('render app...')
  renderTitle(newAppState.title, oldAppState.title)
  renderContent(newAppState.content, oldAppState.content)
}

function renderTitle (newTitle, oldTitle = {}) {
  if (newTitle === oldTitle) return // 数据没有变化就不渲染了
  console.log('render title...')
  const titleDOM = document.getElementById('title')
  titleDOM.innerHTML = newTitle.text
  titleDOM.style.color = newTitle.color
}

function renderContent (newContent, oldContent = {}) {
  if (newContent === oldContent) return // 数据没有变化就不渲染了
  console.log('render content...')
  const contentDOM = document.getElementById('content')
  contentDOM.innerHTML = newContent.text
  contentDOM.style.color = newContent.color
}

let appState = {
  title: {
    text: 'React.js 小书',
    color: 'red',
  },
  content: {
    text: 'React.js 小书内容',
    color: 'blue'
  }
}

function stateChanger (state, action) {
  switch (action.type) {
    case 'UPDATE_TITLE_TEXT':
      return { // 构建新的对象并且返回
        ...state,
        title: {
          ...state.title,
          text: action.text
        }
      }
    case 'UPDATE_TITLE_COLOR':
      return { // 构建新的对象并且返回
        ...state,
        title: {
          ...state.title,
          color: action.color
        }
      }
    default:
      return state // 没有修改，返回原来的对象
  }
}

const store = createStore(appState, stateChanger)
let oldState = store.getState() // 缓存旧的 state
store.subscribe(() => {
  const newState = store.getState() // 数据可能变化，获取新的 state
  renderApp(newState, oldState) // 把新旧的 state 传进去渲染
  oldState = newState // 渲染完以后，新的 newState 变成了旧的 oldState，等待下一次数据变化重新渲染
})

renderApp(store.getState()) // 首次渲染页面
store.dispatch({ type: 'UPDATE_TITLE_TEXT', text: '《React.js 小书》' }) // 修改标题文本
store.dispatch({ type: 'UPDATE_TITLE_COLOR', color: 'blue' }) // 修改标题颜色
```

##### 1.5 reducer

由 `stateChanger` 修改而来，是一个纯函数

先把 `appState` 和 `stateChanger` 合并，让 `stateChanger` 有初始化 `appState` 的作用，并修改`stateChanger`  函数名为 `reducer`

```js
function reducer (state, action) {
  	// 初始化数据
	if (!state) {
    return {
      title: {
        text: 'React.js 小书',
        color: 'red',
      },
      content: {
        text: 'React.js 小书内容',
        color: 'blue'
      }
    }
  }
  switch (action.type) {
    case 'UPDATE_TITLE_TEXT':
      return {
        ...state,
        title: {
          ...state.title,
          text: action.text
        }
      }
    case 'UPDATE_TITLE_COLOR':
      return {
        ...state,
        title: {
          ...state.title,
          color: action.color
        }
      }
    default:
      return state
  }
}
```

`createStore` 里只需要传入 `stateChanger` 函数，

```js
function createStore (reducer) {
  let state = null
  const listeners = []
  const subscribe = (listener) => listeners.push(listener)
  const getState = () => state
  const dispatch = (action) => {
    state = reducer(state, action)
    listeners.forEach((listener) => listener())
  }
  dispatch({}) // 初始化 state
  return { getState, dispatch, subscribe }
}
```

##### 1.6 `createStore` 模板

```js
// 定一个 reducer
function reducer (state, action) {
  /* 初始化 state 和 switch case */
}

// 生成 store
const store = createStore(reducer)

// 监听数据变化重新渲染页面
store.subscribe(() => renderApp(store.getState()))

// 首次渲染页面
renderApp(store.getState()) 

// 后面可以随意 dispatch 了，页面自动更新
store.dispatch(...)
```



#### 2.React-Redux

##### 2.0 前置知识

如果一个组件的渲染只依赖于外界传进去的 `props` 和自己的 `state`，而并不依赖于其他的外界的任何数据，也就是说像纯函数一样，给它什么，它就吐出（渲染）什么出来。这种组件的复用性是最强的，别人使用的时候根本不用担心任何事情，只要看看 `PropTypes` 它能接受什么参数，然后把参数传进去控制它就行了。

我们把这种组件叫做 Pure Compoent，因为它就像纯函数一样，可预测性非常强，对参数（`props`）以外的数据零依赖，也不产生副作用。这种组件也叫 Dumb Component，因为它们呆呆的，让它干啥就干啥。写组件的时候尽量写 Dumb Component 会提高我们的组件的可复用性。

##### 2.1 Connect

用高阶组件帮助我们从 `context` 中取数据，通过 `props` 传给 `Dumb` 组件。这个高阶组件起名叫 `connect`

```js
export connect = (WrappedComponent) => {
  class Connect extends Component {
    static contextTypes = {
      store: PropTypes.object
    }

    // TODO: 从 store 取数据

    render () {
      return <WrappedComponent />
    }
  }

  return Connect
}
```

`connect` 函数接受一个 `WrappedComponent` 组件，再把这个组件包在一个新组件  `Connect` 中返回。新组件 `Connect` 获取 `store` 中的数据，通过 `props` 传给  `WrappedComponent` 。

##### 2.2 mapStateToprops

不同的  `WrappedComponent` 需要不同的数据，这要求我们告知高阶组件 `connect` 特定的  `WrappedComponent` 需要什么数据，所以我们可以另外传给 `connect` 一个函数 `mapStateToprops`。

```js
const mapStateToProps = (state) => {
  return {
    themeColor: state.themeColor,
    themeName: state.themeName,
    fullName: `${state.firstName} ${state.lastName}`
    ...
  }
}
```

现在 `connect` 可以写为

```js
export const connect = (mapStateToProps) => (WrappedComponent) => {
  class Connect extends Component {
    static contextTypes = {
      store: PropTypes.object
    }

    render () {
      const { store } = this.context
      let stateProps = mapStateToProps(store.getState())
      // {...stateProps} 意思是把这个对象里面的属性全部通过 `props` 方式传递进去
      return <WrappedComponent {...stateProps} />
    }
  }

  return Connect
}
```

`connect`用法：

```js
const mapStateToProps = (state) => {
  return {
    themeColor: state.themeColor
  }
}
Header = connect(mapStateToProps)(Header)
```

##### 2.3  mapDispatchToProps

`connect` 还需要知道组件需要如何触发 `dispatch`，` mapDispatchToProps` 就是为了满足这个需求的。和 `mapStateToProps` 一样，它返回一个对象，这个对象内容会同样被 `connect` 当作是 `props` 参数传给被包装的组件。不一样的是，这个函数不是接受 `state` 作为参数，而是 `dispatch`，你可以在返回的对象内部定义一些函数，这些函数会用到 `dispatch` 来触发特定的 `action`。

```js
const mapDispatchToProps = (dispatch) => {
  return {
    onSwitchColor: (color) => {
      dispatch({ type: 'CHANGE_COLOR', themeColor: color })
    }
  }
}
```



##### 2.4 Provider

`connect` 组件需要拿到 `store` 才能传给子组件，所以我们增加一个组件叫做 `Provider`, 专门用于存放 `store`。

```js
export class Provider extends React.Component{
    static propTypes = {
        store: PropTypes.object,
        children: PropTypes.any
    }
    static childContextTypes = {
        store: PropTypes.object
      }
      getChildContext(){
        return { store }
      }
    render(){
        <div>{this.props.children}</div>
    }
}
```

`Provider` 是一个容器组件，会把嵌套的内容原封不动作为自己的子组件渲染出来。它还会把外界传给它的 `props.store` 放到 context，这样子组件 `connect` 的时候都可以获取到。

