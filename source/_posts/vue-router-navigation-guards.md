---
title: 【Vue】导航守卫（路由钩子）
date: 2023-03-26 22:13:40
categories: Vue
tags:
- Vue
- vue-router
---
## 概念

“导航”表示路由正在发生变化

vue-router 提供的导航守卫主要用来通过跳转或取消的方式守卫导航。有多种机会植入路由导航过程 中：全局的, 单个路由独享的, 或者组件级的。

<!-- more -->

导航守卫：包括**全局导航守卫**和**局部导航守卫**

路由钩子函数有三种：

- 全局守卫： beforeEach（全局前置守卫）、beforeResolve（全局解析守卫）、 afterEach（全局后置钩子）
- 路由独享守卫(单个路由里面的钩子)： beforeEnter
- 组件守卫：beforeRouteEnter、 beforeRouteUpdate、 beforeRouteLeave



## **全局守卫**

vue-router全局有三个守卫

- router.beforeEach：全局前置守卫，进入路由之前
- router.beforeResolve：全局解析守卫，在beforeRouteEnter调用之后调用（不常用）
- router.afterEach ：全局后置钩子，进入路由之后

### 全局前置守卫：beforeEach

你可以使用 router.beforeEach 注册一个全局前置守卫：

```js
const router = new VueRouter({ ... })
router.beforeEach((to, from, next) => {
  // to和from都是路由实例
  // to：即将跳转到的路由
  // from：现在的要离开的路由
  // next：函数
})
```

1. next: Function : 一定要调用该方法来 **resolve** 这个钩子。执行效果依赖 next 方法的调用参数。
2. next() : 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 **confirmed** (确认的)。
3. next(false) : 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 from 路由对应的地址。
4. next('/') **或者** next({ path: '/' }) : 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 next 传递任意位置对象，且允许设置诸如 replace: true 、 name: 'home' 之类的选项以及任何用在 router-link 的 to prop 或 router.push 中的选项。
5. next(error) : (2.4.0+) 如果传入 next 的参数是一个 Error 实例，则导航会被终止且该错误会 被传递给 router.onError() 注册过的回调。

注意：如果是next('/') 或者 next({ path: '/' })，只要带了要放行的路径，那么前面必须有判断，在什么时候给他放行，不然他会一直循环。

### 全局解析守卫：beforeResolve

2.5.0 新增

```js
// 全局解析守卫 
router.beforeResolve((to, from, next) => {})
```

在 2.5.0+ 你可以用 router.beforeResolve 注册一个全局守卫。这和 router.beforeEach 类似，区别是在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析之后**，解析守卫就被调用。

### 全局后置钩子：afterEach

你也可以注册全局后置钩子，然而和守卫不同的是，这些钩子不会接受 next 函数也不会改变导航本身：

```js
// 全局后置钩子
router.afterEach((to,form) => {})
```

因为：afterEach被调用时，路由已经跳转完成，所以不需要next函数



## 路由独享守卫：beforeEnter

如果不想在全局配置路由的话，可以为某些路由单独配置守卫

比如：给mainpage页面单独配置守卫

```js
{ 
  path: '/mainpage', 
  name: 'About', 
  component: About, // 路由独享守卫 
  beforeEnter: (to,from,next) => {
    if (from.name === '/mainpage/about') {
      alert("这是从about来的") 
    } else {
      alert("这不是从about来的") 
    }
    next(); // 必须调用来进行下一步操作。否则是不会跳转的
  } 
}
```



## 组件守卫

最后，你可以在路由组件内直接定义以下路由导航守卫：

- beforeRouteEnter()：进入路由前
- beforeRouteUpdate()：路由复用同一个组件时
- beforeRouteLeave()：离开当前路由时

在Product中举个例子

```js
// 全局解析守卫
router.beforeResolve((to,from.next) => {})
// 全局后置钩子
router.afterEach((to,form) => {})

{
  path: '/mainpage',
  name: 'About',
  component: About,
  // 路由独享守卫
  beforeEnter:(to,from,next) => {
    if(from.name === '/mainpage/about'){
      alert("这是从about来的")
    }else{
      alert("这不是从about来的")
    }
    next(); // 必须调用来进行下一步操作。否则是不会跳转的
  }
},
  
export default {
  /* 
    组件内守卫beforeRouteUpdate被触发的条件是：当前路由改变，但是该组件被复用的时候。
    比如说：product/orders到product/cart这个路由，都复用了 Product.vue 这个组件，这个时候
    beforeRouteUpdate就会被触发。可以获取到this实例。
  */
  beforeRouteEnter (to, from, next) {
    console.log(to.name);
    // 因为这个钩子调用的时候，组件实例还没有被创建出来，因此获取不到this
    // 如果想获取到实例的话
    // next(vm => {
    //   // 这里的vm是组件的实例（this）
    // });
    next();
  },
  beforeRouteUpdate(to,from,next){
    console.log(to.name, from.name);
    next();
  },
  // 路由即将要离开的时候调用此方法
  // 比如说，用户编辑了一个东西，但是还么有保存，这时候他要离开这个页面，就要提醒他一下，还没保存，是否要离开
  beforeRouteLeave (to, from, next) {
    const leave = confirm("确定要离开吗？");
    if (leave) {
      next() // 离开
    } else {
      next(false) // 不离开
    }
  }
}
```

- beforeRouteUpdate被触发的条件是：当前路由改变，但是该组件被复用的时候。

  比如说：product/orders到product/cart这个路由，都复用了 Product.vue 这个组件，这个时候

  beforeRouteUpdate就会被触发。可以获取到this实例。

  

## 一个完整的导航解析流程

1、导航被触发。

2、在失活的组件（即将离开的页面组件）里调用离开守卫。 beforeRouteLeave

3、调用全局的 beforeEach 守卫。

4、在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。

5、在路由配置里调用（路由独享的守卫） beforeEnter。

6、解析异步路由组件

7、在被激活的组件（即将进入的页面组件）里调用 beforeRouteEnter。

8、调用全局的 beforeResolve 守卫 (2.5+)。

9、导航被确认。

10、调用全局的 afterEach 钩子。所有的钩子都触发完了。

11、触发 DOM 更新。

12、用创建好的实例调用 beforeRouteEnter 守卫中传给 next 的回调函数。