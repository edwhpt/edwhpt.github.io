---
title: 【JS】全局捕获 Promise 异常
date: 2023-03-26 21:08:34
categories: JavaScript
tags:
- JS
- Promise
---

## JS 事件监听

#### 全局捕获示例：unhandledrejection

```javascript
window.addEventListener('unhandledrejection', function(event) {
  // the event object has two special properties:
  alert(event.promise); // [object Promise] - the promise that generated the error
  alert(event.reason); // Error: Whoops! - the unhandled error object
});

new Promise(function() {
  throw new Error("Whoops!");
}); // no catch to handle the error
```

<!-- more -->

#### 简单封装：

```js
let myPromise = (promise, callback) => {
  promise.then(callback).catch(console.log)
}
let p = new Promise((resolve, reject) => reject('error'))
let cb = () => console.log('callback')
myPromise(p, cb)  // error
```

## Vue 钩子函数

### vue2：errorHandler

在 Vue2 的全局配置中提供了一个 `errorHandler` 钩子可以用于捕获全局异常，但是最低版本要求 2.2.0+

`errorHandler` 第一个参数 `err` 是具体的错误信息，第二个参数 `vm` 是 Vue 组件信息，第三个参数 `info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子。一般为了捕获 Vue 特定的 `info` 信息，在内部处理时还会加上一层 `nextTick` ，确保捕获的是 DOM 渲染完成之后的信息。另外最好在根据不同环境配置判断是否需要捕获异常，增加程序的灵活性

```js
// errorHandler 使用示例
import Vue from 'vue'

// 配置项形式：'development' | ['development', 'production']
const { errorLog: needErrorLog } = settings

// 根据配置判断什么环境下需要捕获异常
function checkNeedErrorLog() {
  const env = process.env.NODE_ENV

  if (isString(needErrorLog)) {
    return env === needErrorLog
  }
  if (isArray(needErrorLog)) {
    return needErrorLog.includes(env)
  }

  return false
}

// 全局异常捕获
if (checkNeedErrorLog()) {
  Vue.config.errorHandler = function (err, vm, info) {
    Vue.nextTick(() => {
      console.error(`[${projectName}]: ${err}。`, `Vue info: ${info}`, vm)
    })
  }
}
```

根据[官网的描述](https://link.juejin.cn/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fapi%2F%23errorHandler)，不同的 Vue 版本捕获的信息不同，所以建议最好是更新 Vue 2.6.0 以上的版本，这样就可以全局捕获到 Promise 和 async / await 抛出的异常了

### vue3：warnHandler

在 Vue3 中，除了提供 `errorHandler` 钩子外，还提供了 `warnHandler` 钩子，两个钩子的用法相同，区别是是 `warnHandler` 只在开发环境生效，生产环境会被忽略

```js
app.config.warnHandler = function(msg, vm, trace) {
  // `trace` 是组件的继承关系追踪
}
```

