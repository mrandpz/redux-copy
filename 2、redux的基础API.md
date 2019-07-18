## Action

Q：为什么需要把type 定义成常量？

A：其实也可以直接传字符串，弄成常量的目的是防止其他地方也需要用到这个常量（其实基本上不会用到这个常量）
```
代码查看model下的actions
```

## Reducer 

Reducer 是个 <b>纯函数</b> 接收旧的state和action，返回新的state
reducer中不要做：  
· 修改传入参数  
· 执行有副作用的操作，比如API请求或者路由跳转
· 调用非纯函数，如Date.now(),Math.random()

注意点：
```js
不要修改state，如果用了Object.assign，要创建一个空对象去接收拷贝后的值，其实开发中基本都是用扩展运算符。

在default的时候返回旧的state
```

多个reducer的时候用combineReducer
```js
import {combineReducers} from 'redux'

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```
相当于：
```js
export default function todoApp(state={},action){
  return {
    visibilityFilter:visibilityFilter(state.visibilityFilter,action),
    todos: todos(state.todos,action)
  }
}
```

## store
```js
let store = createStore(todoApp)

// 打印初始状态
console.log(store.getState())

// 每次 state 更新时，打印日志
// 注意 subscribe() 返回一个函数用来注销监听器
const unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

// 发起一系列 action
store.dispatch(addTodo('Learn about actions'))
store.dispatch(addTodo('Learn about reducers'))
store.dispatch(addTodo('Learn about store'))
store.dispatch(toggleTodo(0))
store.dispatch(toggleTodo(1))
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED))

// 停止监听 state 更新
unsubscribe();
```





