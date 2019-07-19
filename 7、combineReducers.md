一样的先看代码解构,传参和返回值，传入Object对象，返回一个combination函数

```js
export default function combineReducers(reducers) {
  ... 略
  return function combination(state = {}, action) {
    ...略
  }
}
```


```js
// 做一个浅拷贝
const reducerKeys = Object.keys(reducers)
const finalReducers = {}

for (let i = 0; i < reducerKeys.length; i++) {
  const key = reducerKeys[i]

  if (process.env.NODE_ENV !== 'production') {
    if (typeof reducers[key] === 'undefined') {
      warning(`No reducer provided for key "${key}"`)
    }
  }

  if (typeof reducers[key] === 'function') {
    finalReducers[key] = reducers[key]
  }
}
const finalReducerKeys = Object.keys(finalReducers)
```

## 2
```js
function assertReducerShape(reducers) {
  Object.keys(reducers).forEach(key => {
    const reducer = reducers[key]
    const initialState = reducer(undefined, { type: ActionTypes.INIT })

    if (typeof initialState === 'undefined') {
      throw new Error(
        `Reducer "${key}" returned undefined during initialization. ` +
          `If the state passed to the reducer is undefined, you must ` +
          `explicitly return the initial state. The initial state may ` +
          `not be undefined. If you don't want to set a value for this reducer, ` +
          `you can use null instead of undefined.`
      )
    }

    if (
      typeof reducer(undefined, {
        type: ActionTypes.PROBE_UNKNOWN_ACTION()
      }) === 'undefined'
    ) {
      throw new Error(
        `Reducer "${key}" returned undefined when probed with a random type. ` +
          `Don't try to handle ${ActionTypes.INIT} or other actions in "redux/*" ` +
          `namespace. They are considered private. Instead, you must return the ` +
          `current state for any unknown actions, unless it is undefined, ` +
          `in which case you must return the initial state, regardless of the ` +
          `action type. The initial state may not be undefined, but can be null.`
      )
    }
  })
}


// This is used to make sure we don't warn about the same
// keys multiple times.
let unexpectedKeyCache
if (process.env.NODE_ENV !== 'production') {
  unexpectedKeyCache = {}
}

let shapeAssertionError
try {
  // 价差reducer是否有默认值
  assertReducerShape(finalReducers)
} catch (e) {
  shapeAssertionError = e
}
```

看返回的函数combination
```js
// 定义一个变量
let hasChanged = false
const nextState = {}
for (let i = 0; i < finalReducerKeys.length; i++) {
  const key = finalReducerKeys[i]
  const reducer = finalReducers[key]
  const previousStateForKey = state[key]
  const nextStateForKey = reducer(previousStateForKey, action)
  if (typeof nextStateForKey === 'undefined') {
    const errorMessage = getUndefinedStateErrorMessage(key, action)
    throw new Error(errorMessage)
  }
  nextState[key] = nextStateForKey
  // 判断state是否发生了变化
  hasChanged = hasChanged || nextStateForKey !== previousStateForKey
}
return hasChanged ? nextState : state
```