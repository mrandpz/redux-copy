文件目录结构

```js
│  applyMiddleware.js
│  bindActionCreators.js
│  combineReducers.js
│  compose.js
│  createStore.js
│  index.js
│  
└─utils
        actionTypes.js
        isPlainObject.js
        warning.js
```

先看utils 下有什么内容  
### actionTypes.js
```js
const randomString = () =>
  Math.random()
    .toString(36)
    .substring(7)
    .split('')
    .join('.')

const ActionTypes = {
  INIT: `@@redux/INIT${randomString()}`,
  REPLACE: `@@redux/REPLACE${randomString()}`,
  PROBE_UNKNOWN_ACTION: () => `@@redux/PROBE_UNKNOWN_ACTION${randomString()}`
}

// 对外暴露 ActionTypes 的三个action类型
export default ActionTypes 
```

```js
Math.random()
    .toString(36)

toString(radix) 是可以接受进制参数的,只有Number才有这个进制参数
```

### isPlainObject 判断一个对象是否为简单对象，指的是 obj.__proto__ === Object.prototype,相当于，如果不是 new Object或者对象字面量创建的对象都不是简单对象

```js
export default function isPlainObject(obj) {
  // 如果是null或者类型不是object
  if (typeof obj !== 'object' || obj === null) return false

  let proto = obj

  // 查找 proto 的原型对象，直到不等于null，因为等于null的时候已经是原型链顶端了
  while (Object.getPrototypeOf(proto) !== null) {
    proto = Object.getPrototypeOf(proto)
  }

  // proto 为Object对象，至于为什么用while这么找，是因为防止有人窜改了Object.prototype 
  return Object.getPrototypeOf(obj) === proto
}
```

### warning.js 打印信息
```js
export default function warning(message) {
  /* eslint-disable no-console */
  // 纯粹是为了兼容IE8，对于新时代的人类没啥用
  if (typeof console !== 'undefined' && typeof console.error === 'function') {
    console.error(message)
  }
  /* eslint-enable no-console */
  try {
    // This error was thrown as a convenience so that if you enable
    // "break on all exceptions" in your console,
    // it would pause the execution at this line.
    throw new Error(message)
  } catch (e) {} // eslint-disable-line no-empty
}
```


