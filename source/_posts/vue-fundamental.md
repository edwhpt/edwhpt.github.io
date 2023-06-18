---
title: 【Vue】基本原理
date: 2023-03-26 22:39:44
categories: Vue
tags: 
- vue
---
## Vue响应式原理
当一个Vue实例创建时，vue会遍历data选项的属性，用Object.defineProperty（vue3.0使用proxy）将它们转为 getter/setter 并且在内部追踪相关依赖，在属性被访问和修改时通知变化。每个组件实例都有相应的watcher程序实例，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的setter被调用时，会通知watcher重新计算，从而致使它关联的组件得以更新。

<!-- more -->

### Vue双向数据绑定

View的变化能实时让Model发生变化，而Modal的变化也能实时更新到View。

Vue采用数据劫持结合发布者-订阅者模式的方式，通过ES5提供的Object.defineProperty()方法来劫持（监控）各个属性的getter、setter，并在数据（对象）发生变动时通知订阅者，触发相应的监听回调。并且，由于是在不同的数据上触发同步，可以精确的将变更发送给绑定的视图，而不是对所有的数据都执行一次检测。
要实现Vue的数据双向绑定，大致可以划分三个模块：Observer、Compile、Watcher。

* Observer 数据监听器，负责对数据对象的所有属性进行监听（数据劫持），监听到数据变化后通知订阅者。
* Compile 指令解析器，扫描模板，并对指令进行解析，然后绑定指定事件。
* Watcher 订阅器，关联Observer和Compile，能够订阅并收到属性变动的通知，执行指令绑定的相应操作，更新视图。

### 版本比较
vue是基于依赖收集的双向绑定；
3.0之前的版本使用Object.defineProperty，3.0版本使用Proxy。

* 基于 数据劫持/依赖收集 的双向绑定的优点：
  * 不需要显示的调用，Vue利用数据劫持+发布订阅，可以直接通知变化并且驱动视图。
  * 直接得到精确的变化数据，劫持了属性的setter，当属性值改变我们可以精确的获取变换的内容newVal，不需要额外的diff操作。

* Object.defineProperty的缺点：
  * 不能监听数组，因为数组没有getter和setter，因为数据长度不确定，如果太长性能负担太大。
  * 只能监听属性，而不是整个对象；需要遍历属性。
  * 只能监听属性变化，不能监听属性的删减。

* Proxy的优点：
  * 可以监听数组。
  * 监听整个对象而不是属性。
  * 13种拦截方法，强大很多。
  * 返回新对象而不是直接修改原对象，更符合immutable。

* Proxy的缺点：
 * 兼容性不好，且无法用polyfill磨平。

### v-model 原理
我们在 vue 项目中主要使用 v-model 指令在表单 input、textarea、select 等元素上创建双向数据绑定，我们知道 v-model 本质上不过是语法糖，v-model 在内部为不同的输入元素使用不同的属性并抛出不同的事件：

* text 和 textarea 元素使用 value 属性和 input 事件；
* checkbox 和 radio 使用 checked 属性和 change 事件；
* select 字段将 value 作为 prop 并将 change 作为事件。

以 input 表单元素为例：

```vue
<input v-model='something'>
```


相当于
```vue
<input v-bind:value="something" v-on:input="something = $event.target.value">
```

如果在自定义组件中，v-model 默认会利用名为 value 的 prop 和名为 input 的事件，如下所示：

父组件：

```vue

<ModelChild v-model="message"></ModelChild>
```

子组件：

```vue
<template>
  <div>{{ value }}</div>
</template>
<script>
export default {
  props:{
      value: String
  },
  methods: {
    test1(){
      this.$emit('input', '小红')
    },
  }
}
</script>
```



## 虚拟DOM 实现原理

* 用 JavaScript 对象模拟真实 DOM 树，对真实 DOM 进行抽象；
* diff 算法 — 比较两棵虚拟 DOM 树的差异；
* pach 算法 — 将两个虚拟 DOM 对象的差异应用到真正的 DOM 树



## 组件中 data 为什么是函数

**为什么组件中的 data 必须是一个函数，然后 return 一个对象，而 new Vue 实例里，data 可以直接是一个对象？**

因为组件是用来复用的，且 JS 里对象是引用关系，如果组件中 data 是一个对象，那么这样作用域没有隔离，子组件中的 data 属性值会相互影响，如果组件中 data 选项是一个函数，那么每个实例可以维护一份被返回对象的独立的拷贝，组件实例之间的 data 属性值不会互相影响；而 new Vue 的实例，是不会被复用的，因此不存在引用对象的问题。

