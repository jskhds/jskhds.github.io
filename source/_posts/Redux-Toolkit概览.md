---
title: Redux-Toolkit概览
date: 2022-02-08 21:28:09
tags: Redux-Toolkit
---

RTK 学习笔记

<!--more-->

#### 1.什么是 `RTK`

一个简化 react-redux 操作的工具。可以让我们简化配置，少装package，少写模板代码。

安装

```powershell
# Redux + Plain JS template
npx create-react-app my-app --template redux

# Redux + TypeScript template
npx create-react-app my-app --template redux-typescript

# npm
npm install @reduxjs/toolkit
# Yarn
yarn add @reduxjs/toolkit
```



#### 2.分析

RTK源码中的 dependencies 包括

```js
// 没有安装 react-redux 说明这并不仅仅是给 react 用的，也可以给 vue 用
immer  //允许用户直接修改 state
redux
redux-thunk  // 默认开启，用来处理异步逻辑
reselect  // 
```



#### 3.api

##### createReducer

一般配置：

```js
const initialState = { value: 0 }

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, value: state.value + 1 }
    case 'decrement':
      return { ...state, value: state.value - 1 }
    case 'incrementByAmount':
      return { ...state, value: state.value + action.payload }
    default:
      return state
  }
}
```

`createReducer`配置:有两种办法，Map Object 和 Builder Callback,在这里介绍前一种方法

```js
//1.直接写
const counterReducer = createReducer(0, {
  increment: (state, action) => state + action.payload,
  decrement: (state, action) => state - action.payload,
})

console.log(counterReducer.getInitialState()) // 0

//2. 配合 createAction一起写
const increment = createAction('increment')
const decrement = createAction('decrement')

const counterReducer = createReducer(0, {
  [increment]: (state, action) => state + action.payload,
  [decrement.type]: (state, action) => state - action.payload
})
```



##### `createSlice`（核心）

一般来说，我们不会特别去单独使用 `createReducer` 和 `createAction`一个函数，用 `createSlice` 比较多。它接受initial state，reducer函数对象，slice name，自动生成对应 reducers 和 state的 action creators 和 action types

```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'

interface CounterState {
  value: number
}

const initialState = { value: 0 } as CounterState

const counterSlice = createSlice({
  name: 'counter',  // slice 名称，相当于分割 store 之后的命名空间
  initialState,  // 数据的初始化
  reducers: {  // 一个特殊的 reducer 把 action 和 state 结合在一起了
    increment(state) {
      state.value++  // state 直接修改 RTK自动加载 immer，immer在最底层重建 state ，所以帮助我们修改了
    },
    decrement(state) {
      state.value--
    },
    incrementByAmount(state, action: PayloadAction<number>) {
      state.value += action.payload
    },
  },
})

export const { increment, decrement, incrementByAmount } = counterSlice.actions
export default counterSlice.reducer
```

example

```ts
import { createSlice, createAction, PayloadAction } from '@reduxjs/toolkit'
import { createStore, combineReducers } from 'redux'

const incrementBy = createAction<number>('incrementBy')
const decrementBy = createAction<number>('decrementBy')

const counter = createSlice({
  name: 'counter',
  initialState: 0 as number,
  reducers: {
    increment: (state) => state + 1,
    decrement: (state) => state - 1,
    multiply: {
      reducer: (state, action: PayloadAction<number>) => state * action.payload,
      prepare: (value?: number) => ({ payload: value || 2 }), // fallback if the payload is a falsy value
    },
  },
  // 解决环形引用
  extraReducers: (builder) => {
    builder.addCase(incrementBy, (state, action) => {
      return state + action.payload
    })
    builder.addCase(decrementBy, (state, action) => {
      return state - action.payload
    })
  },
})

const user = createSlice({
  name: 'user',
  initialState: { name: '', age: 20 },
  reducers: {
    setUserName: (state, action) => {
      state.name = action.payload // mutate the state all you want with immer
    },
  },
  // "map object API"
  extraReducers: {
    // @ts-expect-error in TypeScript, this would need to be [counter.actions.increment.type]
    [counter.actions.increment]: (
      state,
      action /* action will be inferred as "any", as the map notation does not contain type information */
    ) => {
      state.age += 1
    },
  },
})

// 使用 combineReducers 把两个 slice 里面的 reducer 合并起来
const reducer = combineReducers({
  counter: counter.reducer,
  user: user.reducer,
})
// 创建 store
const store = createStore(reducer)

// 然后就可以直接用 store 对象来调用，优点在于把 action 和各种 switch 语句消除了，代码简洁up，还解决了 string类型action出错的可能性

store.dispatch(counter.actions.increment())
// -> { counter: 1, user: {name : '', age: 21} }
store.dispatch(counter.actions.increment())
// -> { counter: 2, user: {name: '', age: 22} }
store.dispatch(counter.actions.multiply(3))
// -> { counter: 6, user: {name: '', age: 22} }
store.dispatch(counter.actions.multiply())
// -> { counter: 12, user: {name: '', age: 22} }
console.log(`${counter.actions.decrement}`)
// -> "counter/decrement" RTK自动生成的 命名空间/对应 reducer
store.dispatch(user.actions.setUserName('eric'))
// -> { counter: 12, user: { name: 'eric', age: 22} }
```

##### `configureStore`

用 `configureStore` 来代替 redux 中原生的 `createStore`

配置

```ts
type ConfigureEnhancersCallback = (
  defaultEnhancers: StoreEnhancer[]
) => StoreEnhancer[]

interface ConfigureStoreOptions<
  S = any,
  A extends Action = AnyAction,
  M extends Middlewares<S> = Middlewares<S>
> {
  /**
   * A single reducer function that will be used as the root reducer, or an
   * object of slice reducers that will be passed to `combineReducers()`.
   */
  reducer: Reducer<S, A> | ReducersMapObject<S, A>

  /**
   * An array of Redux middleware to install. If not supplied, defaults to
   * the set of middleware returned by `getDefaultMiddleware()`.
   */
  middleware?: ((getDefaultMiddleware: CurriedGetDefaultMiddleware<S>) => M) | M

  /**
   * Whether to enable Redux DevTools integration. Defaults to `true`.
   *
   * Additional configuration can be done by passing Redux DevTools options
   */
  devTools?: boolean | DevToolsOptions

  /**
   * The initial state, same as Redux's createStore.
   * You may optionally specify it to hydrate the state
   * from the server in universal apps, or to restore a previously serialized
   * user session. If you use `combineReducers()` to produce the root reducer
   * function (either directly or indirectly by passing an object as `reducer`),
   * this must be an object with the same shape as the reducer map keys.
   */
  preloadedState?: DeepPartial<S extends any ? S : S>

  /**
   * The store enhancers to apply. See Redux's `createStore()`.
   * All enhancers will be included before the DevTools Extension enhancer.
   * If you need to customize the order of enhancers, supply a callback
   * function that will receive the original array (ie, `[applyMiddleware]`),
   * and should return a new array (such as `[applyMiddleware, offline]`).
   * If you only need to add middleware, you can use the `middleware` parameter instead.
   */
  enhancers?: StoreEnhancer[] | ConfigureEnhancersCallback
}

function configureStore<S = any, A extends Action = AnyAction>(
  options: ConfigureStoreOptions<S, A>
): EnhancedStore<S, A>
```

简单使用

```ts
import { configureStore } from '@reduxjs/toolkit'

import rootReducer from './reducers'

const store = configureStore({ reducer: rootReducer })
// The store now has redux-thunk added and the Redux DevTools Extension is turned on
```

##### `configureStore ` 

见第四点

##### `createAsyncThunk`  

见第四点

#### 4.RTK 异步处理数据

##### `configureStore`

```tsx
import { configureStore } from '@reduxjs/toolkit'

import rootReducer from './reducers'  // rootReducer是一系列 reducers 的集合，从 slice中导入

const store = configureStore({ reducer: rootReducer })
// The store now has redux-thunk added and the Redux DevTools Extension is turned on
```



##### `createAsyncThunk`

接受 action type，返回一个标准的redux thunk action creator 以及promise和三个 action：

For example, a `type` argument of `'users/requestStatus'` will generate these action types:

- `pending`: `'users/requestStatus/pending'`
- `fulfilled`: `'users/requestStatus/fulfilled'`
- `rejected`: `'users/requestStatus/rejected'`

使用的时候要确保 store 使用的封装函数是 `configureStore` 而不是 `createStore`

object map 创建 reducer 的方式：

```tsx
import { createSlice, PayloadAction, createAsyncThunk } from "@reduxjs/toolkit";
import axios from "axios";

interface ProductDetailState {
  loading: boolean;
  error: string | null;
  data: any;
}

const initialState: ProductDetailState = {
  loading: true,
  error: null,
  data: null,
};

export const getProductDetail = createAsyncThunk(
  "productDetail/getProductDetail", //命名空间/文件名
  async (touristRouteId: string, thunkAPI) => {
    const { data } = await axios.get(
      /*请求的 api 地址*/
    );
    return data;
  }
);

export const productDetailSlice = createSlice({
  name: "productDetail",
  initialState,
  reducers: {
    
  },
  extraReducers: {
    [getProductDetail.pending.type]: (state) => {
      // return { ...state, loading: true };
      state.loading = true;
    },
    [getProductDetail.fulfilled.type]: (state, action) => {
      state.data = action.payload;
      state.loading = false;
      state.error = null;
    },
    [getProductDetail.rejected.type]: (state, action: PayloadAction<string | null>) => {
      //   const ddd = action.payload;
      state.loading = false;
      state.error = action.payload;
    },
  }
});

```



#### 





