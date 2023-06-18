---
title: 【JS】Event-Loop 事件循环（宏任务/微任务）
date: 2023-03-26 13:26:52
categories: JavaScript
tags: 
- javascript
- event-loop
- promise
- async/await
- set-timeout
---

众所周知，JavaScript 是一门单线程语言，可是浏览器又能很好的处理异步请求，那么到底是为什么呢？

JS 的执行环境一般是浏览器和 Node.js，两者稍有不同，这里只讨论浏览器环境下的情况。

JS 执行过程中会产生两种任务，分别是：同步任务和异步任务。

- 同步任务：比如声明语句、for、赋值等，读取后依据从上到下从左到右，立即执行。
- 异步任务：比如 ajax 网络请求，setTimeout 定时函数等都属于异步任务。异步任务会通过任务队列（Event Queue）的机制（先进先出的机制）来进行协调。

<!-- more -->

### 任务队列（Event Queue）


任务队列中的任务也分为两种，分别是：宏任务（macrotask）和微任务（microtask）

- 宏任务主要包括：scrip（JS 整体代码）、setTimeout、setInterval、setImmediate、I/O、UI 交互
- 微任务主要包括：Promise（重点关注）、process.nextTick（Node.js）、MutaionObserver 

任务队列的执行过程是：先执行一个宏任务，执行过程中如果产出新的宏/微任务，就将他们推入相应的任务队列，之后在执行一队微任务，之后再执行宏任务，如此循环。以上不断重复的过程就叫做 Event Loop（事件循环）。

每一次的循环操作被称为 Tick。
![event-loop flow](/images/js/js-eventloop-flow.jpg)

理解微任务和宏任务的执行执行过程

```js
console.log("script start");

setTimeout(function () {
  console.log("setTimeout");
}, 0);

Promise.resolve()
  .then(function () {
    console.log("promise1");
  })
  .then(function () {
    console.log("promise2");
  });

console.log("script end");
```

按照上面的内容，分析执行步骤：

1. 宏任务：执行整体代码（相当于 script 中的代码）：

   输出: script start
   遇到 setTimeout，加入宏任务队列，当前宏任务队列（setTimeout）
   遇到 promise，加入微任务，当前微任务队列（promise1）
   输出：script end

2. 微任务：执行微任务队列（promise1）

   输出：promise1，then 之后产生一个微任务，加入微任务队列，当前微任务队列（promise2）
   执行 then，输出 promise2

3. 执行渲染操作，更新界面（敲黑板划重点）。 

4. 宏任务：执行 setTimeout

**输出：setTimeout**

### Promise 的执行


new Promise(..) 中的代码，也是同步代码，会立即执行。只有 then 之后的代码，才是异步执行的代码，是一个微任务。

```js
console.log("script start");

setTimeout(function () {
  console.log("timeout1");
}, 10);

new Promise((resolve) => {
  console.log("promise1");
  resolve();
  setTimeout(() => console.log("timeout2"), 10);
}).then(function () {
  console.log("then1");
});

console.log("script end");
```

步骤解析：

- 当前任务队列：微任务: [], 宏任务：[script]
  宏任务：

  输出：script start
  遇到 timeout1，加入宏任务
  遇到 Promise，输出promise1，直接 resolve，将 then 加入微任务，遇到 timeout2，加入宏任务。
  输出：script end
  宏任务第一个执行结束

- 当前任务队列：微任务[then1]，宏任务[timeou1, timeout2]
  微任务：

  执行 then1，输出then1
  微任务队列清空

- 当前任务队列：微任务[]，宏任务[timeou1, timeout2]
  宏任务：

  输出timeout1

- 当前任务队列：微任务[]，宏任务[timeou2]

  微任务：为空跳过
  宏任务：

  输出timeout2


### async/await 的执行


async 和 await 其实就是 Generator 和 Promise 的语法糖。

async 函数和普通 函数没有什么不同，他只是表示这个函数里有异步操作的方法，并返回一个 Promise 对象

翻译过来其实就是：

```js
// async/await 写法
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
// Promise 写法
async function async1() {
  console.log("async1 start");
  Promise.resolve(async2())
    .then(() => console.log("async1 end"));
}
```

看例子：

```js
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
async function async2() {
  console.log("async2");
}
async1();
setTimeout(() => {
  console.log("timeout");
}, 0);
new Promise(function (resolve) {
  console.log("promise1");
  resolve();
}).then(function () {
  console.log("promise2");
});
console.log("script end");
```

步骤解析：

- 当前任务队列：宏任务：[script]，微任务: []
  宏任务：

  输出：async1 start
  遇到 async2，输出：async2，并将 then（async1 end）加入微任务
  遇到 setTimeout，加入宏任务。
  遇到 Promise，输出：promise1，直接 resolve，将 then（promise2）加入微任务
  输出：script end

- 当前任务队列：微任务[async1 end, promise2]，宏任务[timeout]
  微任务：

  输出：async1 end
  async1 end 出队

  输出：promise2
  promise2 出队

  微任务队列清空 

- 当前任务队列：微任务[]，宏任务[timeout]
  宏任务：

  输出：timeout
  timeout 出队，宏任务清空


"任务队列"是一个事件的队列（也可以理解成消息的队列），IO设备完成一项任务，就在"任务队列"中添加一个事件，表示相关的异步任务可以进入"执行栈"了。主线程读取"任务队列"，就是读取里面有哪些事件。

"任务队列"中的事件，除了IO设备的事件以外，还包括一些用户产生的事件（比如鼠标点击、页面滚动等等）。只要指定过回调函数，这些事件发生时就会进入"任务队列"，等待主线程读取。



所谓"回调函数"（callback），就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。

"任务队列"是一个先进先出的数据结构，排在前面的事件，优先被主线程读取。主线程的读取过程基本上是自动的，只要执行栈一清空，"任务队列"上第一位的事件就自动进入主线程。但是，由于存在后文提到的"定时器"功能，主线程首先要检查一下执行时间，某些事件只有到了规定的时间，才能返回主线程。

----JavaScript中没有任何代码是立即执行的，都是进程空闲时尽快执行

 

### setTimerout 并不准确

由上我们已经知道了 setTimeout 是一个宏任务，会被添加到宏任务队列当中去，按顺序执行，如果前面有。

setTimeout() 的第二个参数是为了告诉 JavaScript 再过多长时间把当前任务添加到队列中。

如果队列是空的，那么添加的代码会立即执行；如果队列不是空的，那么它就要等前面的代码执行完了以后再执行。

看代码：

```js
const s = new Date().getSeconds();
console.log("script start");
new Promise((resolve) => {
  console.log("promise");
  resolve();
}).then(() => {
  console.log("then1");
  while (true) {
    if (new Date().getSeconds() - s >= 4) {
      console.log("while");
      break;
    }
  }
});
setTimeout(() => {
  console.log("timeout");
}, 2000);
console.log("script end"); 
```

因为 then 是一个微任务，会先于 setTimeout 执行，所以，虽然 setTimeout 是在两秒后加入的宏任务，但是因为 then 中的在 while 操作被延迟了4s，所以一直推迟到了4s秒后才执行的setTimeout。

所以输出的顺序是：script start、promise、script end、then1。
四秒后输出：while、timeout 

注意：关于 setTimeout 要补充的是，即便主线程为空，0 毫秒实际上也是达不到的。根据 HTML 的标准，最低是 4 毫秒。有兴趣的同学可以自行了解。

<!-- ### 异步渲染策略 -->
<!-- 以 Vue 为例 nextTick -->



### 总结


有个小 tip：从规范来看，microtask 优先于 task 执行，所以如果有需要优先执行的逻辑，放入 microtask 队列会比 task 更早的被执行。

最后的最后，记住，JavaScript 是一门单线程语言，异步操作都是放到事件循环队列里面，等待主执行栈来执行的，并没有专门的异步执行线程。
