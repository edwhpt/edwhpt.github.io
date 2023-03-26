---
title: 【Vue】组件通信
date: 2023-03-26 22:52:04
categories: Vue
tags: 
- Vue
---
## 父子组件传值

- 父传子：通过props

- 子传父：$emit传递，父组件v-on自定义事件接收

<!-- more -->

## 非父子组件数据传递

### Event Bus

EventBus，就是创建一个事件中心，相当于中转站。

通过$emit传递，$on接收

### Vuex 状态管理
Vuex是一个专为Vue.js应用程序开发的状态管理模式。每一个Vuex应用的核心就是store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的状态（state）。

#### 特点：

1. Vuex的状态存储是响应式的。当Vue组件从store中读取状态的时候，若store中的状态发生变化，那么相应的组件也会相应地得到高效更新。
2. 改变 store 中的状态的唯一途径就是显式地提交（commit）mutation。这样使得我们可以方便地跟踪每一个状态的变化。

#### 主要模块：

* State：定义了应用状态的数据结构，可以在这里设置默认的初始状态。
* Getter：允许组件从 Store 中获取数据，mapGetters 辅助函数仅仅是将 Store 中的 Getter 映射到局部计算属性。
* Mutation：是唯一更改 Store 中状态的方法，且必须是同步函数。
* Action：用于提交 Mutation，而不是直接变更状态，可以包含任意异步操作。
* Module：允许将单一的 Store 拆分为多个 Store 且同时保存在单一的状态树中。

持久化控件：vuex-persistedstate

