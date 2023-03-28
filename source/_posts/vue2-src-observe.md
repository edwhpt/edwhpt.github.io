---
title: 【Vue2源码实现】2.数据响应式（observe）
date: 2023-03-27 13:51:30
categories: 
- Vue
- Vue2-Source
tags:
- Vue
- Vue2-Source
---

响应式基本原理就是，在 Vue 的构造函数中，对 options 的 data 进行处理。即在初始化vue实例的时候，对data、props等对象的每一个属性都通过 Object.defineProperty 定义一次，在数据被set的时候，做一些操作，改变相应的视图。

<!-- more -->

### 数据观测

基于 Object.defineProperty 来实现对数组和对象的劫持

```js
import { newArrayProto } from './array'

class Observer {
  constructor(data){
    if (Array.isArray(data)) {
      // 这里我们可以重写可以修改数组本身的方法 7个方法
      // 切片编程：需要保留数组原有的特性，并且可以重写部分方法
      data.__proto__ = newArrayProto
      this.observeArray(data) // 如果数组中放的是对象 可以监控到对象的变化
    } else {
      this.walk(data)
    }
  }
  // 循环对象"重新定义属性",对属性依次劫持，性能差
  walk(data) {
    Object.keys(data).forEach(key => defineReactive(data, key, data[key]))
  }
  // 观测数组
  observeArray(data) {
    data.forEach(item => observe(item))
  }
}

function defineReactive(data,key,value){
  observe(value)  // 深度属性劫持，对所有的对象都进行属性劫持

  Object.defineProperty(data,key,{
    get(){
      return value
    },
    set(newValue){
      if(newValue == value) return
      observe(newValue) // 修改属性之后重新观测，目的：新值为对象或数组的话，可以劫持其数据
      value = newValue
    }
  })
}

export function observe(data) {
  // 只对对象进行劫持
  if(typeof data !== 'object' || data == null){
    return
  }
  return new Observer(data)
}
```

### 重写数组7个变异方法

7个方法是指：push、pop、shift、unshift、sort、reverse、splice。（这七个都是会改变原数组的） 实现思路：面向切片编程！！！

> 不是直接粗暴重写 Array.prototype 上的方法，而是通过原型链继承与函数劫持进行的移花接木。

利用 Object.create(Array.prototype) 生成一个新的对象 newArrayProto，该对象的 **proto** 指向 Array.prototype，然后将我们数组的 **proto** 指向拥有重写方法的新对象 newArrayProto，这样就保证了 newArrayProto 和 Array.prototype 都在数组的原型链上。

```js
arr.__proto__ === newArrayProto；newArrayProto.__proto__ === Array.prototype
```

然后在重写方法的内部使用 Array.prototype[method].call 调用原来的方法，并对新增数据进行劫持观测。

```js
let oldArrayProto = Array.prototype // 获取数组的原型

export let newArrayProto = Object.create(oldArrayProto)

// 找到所有的变异方法
let methods = ['push', 'pop', 'shift', 'unshift', 'reverse', 'sort', 'splice']

methods.forEach(method => {
  // 这里重写了数组的方法
  newArrayProto[method] = function (...args) {
    // args reset参数收集，args为真正数组，arguments为伪数组
    // 内部调用原来的方法，函数的劫持，切片编程
    const result = oldArrayProto[method].call(this, ...args) 

    // 我们需要对新增的数据再次进行劫持
    let inserted
    let ob = this.__ob__

    switch (method) {
      case 'push':
      case 'unshift': // arr.unshift(1,2,3)
        inserted = args
        break
      case 'splice': // arr.splice(0,1,{a:1},{a:1})
        inserted = args.slice(2)
      default:
        break
    }

    if (inserted) {
      // 对新增的内容再次进行观测
      ob.observeArray(inserted)
    }
    return result
  }
})
```

### 增加 `__ob__` 属性

这是一个恶心又巧妙的属性，我们在 Observer 类内部，把 this 实例添加到了响应式数据上。相当于给所有响应式数据增加了一个标识，并且可以在响应式数据上获取 Observer 实例上的方法

```js
class Observer {
  constructor(data) {
    // 给数据加了一个标识,如果数据上有__ob__ 则说明这个属性被观测过了
    // data.__ob__ = this 
    Object.defineProperty(data, '__ob__', {
      value: this,
      enumerable: false, // 将__ob__ 变成不可枚举 （循环的时候无法获取到，防止栈溢出）
    })

    if (Array.isArray(data)) {
      // 这里我们可以重写可以修改数组本身的方法 7个方法
      // 切片编程：需要保留数组原有的特性，并且可以重写部分方法
      data.__proto__ = newArrayProto
      this.observeArray(data) // 如果数组中放的是对象 可以监控到对象的变化
    } else {
      this.walk(data)
    }
  }

}
```

**`__ob__` 有两大用处：**

1. 如果一个对象被劫持过了，那就不需要再被劫持了，要判断一个对象是否被劫持过，可以通过 `__ob__` 来判断

```js
// 数据观测
export function observe(data) {
  // 只对对象进行劫持
  if (typeof data !== 'object' || data == null) {
    return
  }

  // 如果一个对象被劫持过了，那就不需要再被劫持了 
  // (要判断一个对象是否被劫持过，可以在对象上增添一个实例，用实例的原型链来判断是否被劫持过)
  if (data.__ob__ instanceof Observer) {
    return data.__ob__
  }

  return new Observer(data)
}
```

1. 我们重写了数组的7个变异方法，其中 push、unshift、splice 这三个方法会给数组新增成员。此时需要对新增的成员再次进行观测，可以通过 `__ob__` 调用 Observer 实例上的 observeArray 方法

### 注意事项

> Vue 2 是基于 Object.defineProperty 实现的响应式系统 （这个方法是 ES5 中一个无法 shim 的特性，这也就是 Vue 不支持 IE8 以及更低版本浏览器的原因）
>
> Vue3 是基于 Proxy/Reflect 来实现的

1. Object.defineProperty()  无法检测对象和数组新增的属性和数组长度变化
2. Vue 无法检测通过索引直接修改数组元素的操作，这不是 Object.defineProperty 的原因，而是作者尤老师认为性能消耗与带来的用户体验不成正比。因为数组长度可能很大，对数组进行响应式检测会带来很大的性能消耗
