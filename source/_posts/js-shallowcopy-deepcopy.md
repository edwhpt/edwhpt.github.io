---
title: 【JS】浅拷贝和深拷贝
date: 2023-03-26 20:59:17
categories: JavaScript
tags: 
- js
- shallow-copy
- deep-copy
---

浅拷贝和深拷贝都是创建一份数据的拷贝

## 区别

- 浅拷贝和深拷贝都复制了值和地址，都是为了解决引用类型赋值后互相影响的问题。
- 浅拷贝**只进行一层复制**，深层次的引用类型还是共享内存地址，原对象和拷贝对象还是会互相影响。
- 深拷贝就是**无限层级拷贝**，深拷贝后的原对象不会和拷贝对象互相影响。

<!-- more -->

## 实现方式

### 浅拷贝

#### Object.assign

```js
const obj = {}
const newObj = Object.assign({}, obj)
```

#### 数组的 slice() 和 concat()

```js
const arr = []
const newArr = arr.slice(0)
```

```js
const arr = []
const newArr = [].concat(arr)
```

#### 数组静态方法 Array.from()

```js
const arr = []
const newArr = Array.from(arr)
```

#### 扩展运算符

```js
const arr = []
const newArr = [...arr]
```



### 深拷贝

#### 字符串转换：JSON.stringify() / JSON.parse()

```js
const obj = {}
const newObj = JSON.parse(JSON.stringify(obj))
```

##### 问题：

- 会忽略`undefined`、`symbol`和`函数`
- `NaN`、`Infinity`、`-Infinity` 会被序列化为 `null`
- 不能解决循环引用的问题

#### JS新语法：[structuredClone()](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone)

```js
const obj = {}
const newObj = structuredClone(obj)
```

##### 问题：

- `Function` 类型会报错
- 兼容性差

#### 使用递归手动实现

##### 简单版本

```js
function deepClone (target) {
  if (typeof target !== 'object') {
    // 如果是原始数据类型，直接返回
    return target
  }
  const cloneTarget = {} // 定义克隆对象
  // 遍历原对象
  for (const key in target) {
    // 递归拷贝
    cloneTarget[key] = deepClone(target[key])
  }
 	return cloneTarget // 返回克隆对象
}
```

##### 处理数组、日期、正则、null

```js
function deepClone (target) {
  if (target === null) return target // 处理null
  if (target instanceof Date) return new Date(target) // 处理日期
  if (target instanceof RegExp) return new RegExp(target) // 处理正则
  if (typeof target !== 'object')  return target // 处理原始数据类型
  
  const cloneTarget = new target.constructor() // 定义克隆对象（数组）
 
  for (const key in target) {
    cloneTarget[key] = deepClone(target[key]) // 递归拷贝
  }
 	return cloneTarget // 返回克隆对象
}
```



### 第三方工具库（lodash）

#### 浅拷贝：clone()

#### 深拷贝：cloneDeep()

```js
import {clone, cloneDeep} from 'lodash'

const obj = {}
const cloneObj = clone(obj)
const cloneDeepObj = cloneDeep(obj)
```



## 扩展资料

- [Lodash是如何实现深拷贝的](https://juejin.cn/post/6844903775283445767#heading-5)

- [如何写出一个惊艳面试官的深拷贝?](https://juejin.cn/post/6844903929705136141#heading-7)
