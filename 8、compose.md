compose 的代码比较少，就是把所有的funs传进去，从右往左执行
```js
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}

```

```js
funcs.reduce((a, b) => (...args) => a(b(...args)))
相当于

reduce 的参数 初始值,当前元素，索引，当前元素
funcs.reduce(function(a,b){
  return function(...args){
    return a(b(...args))
  }
})
```