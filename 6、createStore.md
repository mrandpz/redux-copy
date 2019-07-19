```js
...略

先看下整个函数结构，接收三个参数 reducer，preloadedState（不常用，它代表着初始状态），enhancer（中文名字叫增强器）比如：

export default function createStore(reducer, preloadedState, enhancer) {
  ... 略
  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```

看下增强器的代码,有传值的时候必须是个function

```js
if (typeof enhancer !== 'undefined') {
  if (typeof enhancer !== 'function') {
    throw new Error('Expected the enhancer to be a function.')
  }

  return enhancer(createStore)(reducer, preloadedState)
}

···略

function enhancer(createStore) {
    return (reducer,preloadedState) => {
         //逻辑代码
        .......
    }
 }

 ---使用增强器 applyMiddleware(thunk)
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers/index';

const store = createStore(
  rootReducer,
  applyMiddleware(thunk)
);
```

这里需要注意，我们第二个参数是preloadedState，但是我们传入的是enchancer ，这样不会报错？
看看这段代码：

```js

// 如果 preloadedState 是个function，因为enhancer必须是个function，并且enhancer没传值，那么enhancer等于preloadedState，preloadedState重置为undefined。
if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
  enhancer = preloadedState
  preloadedState = undefined
}
```


代码接着往下看：定义了五个变量

```js
let currentReducer = reducer // 传入的reducer
let currentState = preloadedState // 传入的初始值
let currentListeners = [] // 当前订阅者列表
let nextListeners = currentListeners // 新的订阅者列表
let isDispatching = false // 作为一个锁来使用，防止两个action同时修改数据，每次
```


查看函数dispatch，传入action，返回action
```js
function dispatch(action) {
  // 判断action是否为简单对象
  if (!isPlainObject(action)) {
    throw new Error(
      'Actions must be plain objects. ' +
        'Use custom middleware for async actions.'
    )
  }

  // 判断action.type是否存在
  if (typeof action.type === 'undefined') {
    throw new Error(
      'Actions may not have an undefined "type" property. ' +
        'Have you misspelled a constant?'
    )
  }

  // 判断是否有其他的reducer在执行
  if (isDispatching) {
    throw new Error('Reducers may not dispatch actions.')
  }

  try {
    // isDispatching设置为true，防止后续的action来触发reducer操作
    isDispatching = true
    currentState = currentReducer(currentState, action)
  } finally {
    // 执行完成之后才允许触发action
    isDispatching = false
  }

  // 通知各位订阅者做更新操作
  const listeners = (currentListeners = nextListeners)
  for (let i = 0; i < listeners.length; i++) {
    const listener = listeners[i]
    listener()
  }

  return action
}
```

getState,代码很简单，就是返回currentState，如果是在进行reducer操作的时候，无法取值，
store通过getState得出的state是可以直接被更改的，但是redux不允许这么做，因为这样不会通知订阅者更新数据。

```js
function getState() {
  if (isDispatching) {
    throw new Error(
      'You may not call store.getState() while the reducer is executing. ' +
        'The reducer has already received the state as an argument. ' +
        'Pass it down from the top reducer instead of reading it from the store.'
    )
  }

  return currentState
}
```

查看下订阅函数
```js
function ensureCanMutateNextListeners() {
  // 如果两个引用相等，就做一个浅拷贝
  if (nextListeners === currentListeners) {
    nextListeners = currentListeners.slice()
  }
}
```
```js
  // 返回一个取消订阅的函数
  function subscribe(listener) {
    // 是否为函数
    if (typeof listener !== 'function') {
      throw new Error('Expected the listener to be a function.')
    }

  // 是否有reducer在修改
    if (isDispatching) {
      throw new Error(
        'You may not call store.subscribe() while the reducer is executing. ' +
          'If you would like to be notified after the store has been updated, subscribe from a ' +
          'component and invoke store.getState() in the callback to access the latest state. ' +
          'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
      )
    }

    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() {
      // 是否已经取消订阅
      if (!isSubscribed) {
        return
      }

      // 是否有reducer正在修改数据
      if (isDispatching) {
        throw new Error(
          'You may not unsubscribe from a store listener while the reducer is executing. ' +
            'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
        )
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }
```

replaceReducer用来替换reducer
```js
function replaceReducer(nextReducer) {
  if (typeof nextReducer !== 'function') {
    throw new Error('Expected the nextReducer to be a function.')
  }

  currentReducer = nextReducer

  // This action has a similiar effect to ActionTypes.INIT.
  // Any reducers that existed in both the new and old rootReducer
  // will receive the previous state. This effectively populates
  // the new state tree with any relevant data from the old one.
  dispatch({ type: ActionTypes.REPLACE })
}
```

最后一个函数 
```js
 function observable() {
    const outerSubscribe = subscribe
    return {
      /**
       * The minimal observable subscription method.
       * @param {Object} observer Any object that can be used as an observer.
       * The observer object should have a `next` method.
       * @returns {subscription} An object with an `unsubscribe` method that can
       * be used to unsubscribe the observable from the store, and prevent further
       * emission of values from the observable.
       */
      subscribe(observer) {
        if (typeof observer !== 'object' || observer === null) {
          throw new TypeError('Expected the observer to be an object.')
        }

        function observeState() {
          if (observer.next) {
            observer.next(getState())
          }
        }

        observeState()
        const unsubscribe = outerSubscribe(observeState)
        return { unsubscribe }
      },

      [$$observable]() {
        return this
      }
    }
  }
```
