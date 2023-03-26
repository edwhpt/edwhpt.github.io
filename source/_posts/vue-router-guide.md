---
title: vue-router 使用指南
date: 2023-03-26 21:41:55
categories: Vue
tags:
- vue
- vue-router
---

## vue-router 是什么

这里的路由并不是指我们平时所说的硬件路由器，**这里的路由就是SPA（单页应用）的路径管理器**。再通俗的说，vue-router就是WebApp的链接路径管理系统。
<!-- more -->
vue-router是Vue.js官方的路由插件，它和vue.js是深度集成的，适合用于构建单页面应用。vue的单页面应用是基于路由和组件的，路由用于设定访问路径，并将路径和组件映射起来。传统的页面应用，是用一些超链接来实现页面切换和跳转的。在vue-router单页面应用中，则是路径之间的切换，也就是组件的切换。**路由模块的本质 就是建立起url和页面之间的映射关系**。
至于我们为啥不能用a标签，这是因为用Vue做的都是单页应用，就相当于只有一个主的index.html页面，所以你写的<a></a>标签是不起作用的，你必须使用vue-router来进行管理。

## vue-router 实现原理

SPA(single page application):单一页面应用程序，只有一个完整的页面；它在加载页面时，不会加载整个页面，而是只更新某个指定的容器中内容。**单页面应用(SPA)的核心之一是: 更新视图而不重新请求页面**;vue-router在实现单页面前端路由时，提供了两种方式：Hash模式和History模式；根据mode参数来决定采用哪一种方式。

### Hash模式：

**vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。** hash（#）是URL 的锚点，代表的是网页中的一个位置，单单改变#后的部分，浏览器只会滚动到相应位置，不会重新加载网页，也就是说 #是用来指导浏览器动作的，对服务器端完全无用，HTTP请求中也不会不包括#；同时每一次改变#后的部分，都会在浏览器的访问历史中增加一个记录，使用”后退”按钮，就可以回到上一个位置；所以说**Hash模式通过锚点值的改变，根据不同的值，渲染指定DOM位置的不同数据**

### History模式：

由于hash模式会在url中自带#，如果不想要很丑的 hash，我们可以用路由的 history 模式，只需要在配置路由规则时，加入"mode: 'history'",这种模式充分利用 history.pushState API 来完成 URL 跳转而无须重新加载页面。

```js
//main.js文件中
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

当你使用 history 模式时，URL 就像正常的 url，例如 [yoursite.com/user/id](https://link.juejin.cn?target=http%3A%2F%2Fyoursite.com%2Fuser%2Fid)，比较好看！
不过这种模式要玩好，还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 [oursite.com/user/id](https://link.juejin.cn?target=http%3A%2F%2Foursite.com%2Fuser%2Fid) 就会返回 404，这就不好看了。
所以呢，**你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。**

```js
export const routes = [ 
  {path: "/", name: "homeLink", component:Home}
  {path: "/register", name: "registerLink", component: Register},
  {path: "/login", name: "loginLink", component: Login},
  {path: "*", redirect: "/"}
]
```

此处就设置如果URL输入错误或者是URL 匹配不到任何静态资源，就自动跳到到Home页面

### 使用路由模块来实现页面跳转的方式

方式1：直接修改地址栏

方式2：this.$router.push(‘路由地址’)

方式3：`<router-link to="路由地址"></router-link>`



## vue-router 使用方式

- 下载 `npm i vue-router -S` 

- 在main.js中引入 `import VueRouter from 'vue-router';` 

- 安装插件 `Vue.use(VueRouter);` 

- 创建路由对象并配置路由规则

  ```js
  let router = new VueRouter({routes:[{path:'/home',component:Home}]});
  ```

- 将其路由对象传递给Vue的实例，options中加入 `router:router` 

- 在app.vue中留坑 `<router-view></router-view>` 

具体实现请看如下代码：

```js
//main.js文件中引入
import Vue from 'vue';
import VueRouter from 'vue-router';
//主体
import App from './components/app.vue';
import Home from './components/home.vue'
//安装插件
Vue.use(VueRouter); //挂载属性
//创建路由对象并配置路由规则
let router = new VueRouter({
    routes: [
        //一个个对象
        { path: '/home', component: Home }
    ]
});
//new Vue 启动
new Vue({
    el: '#app',
    //让vue知道我们的路由规则
    router: router, //可以简写router
    render: c => c(App),
})
```

最后记得在在app.vue中“留坑”

```vue
//app.vue中
<template>
    <div>
        <!-- 留坑，非常重要 -->
        <router-view></router-view>
    </div>
</template>
<script>
    export default {
        data(){
            return {}
        }
    }
</script>
```



## vue-router 核心要点

### vue-router如何参数传递

**①用name传递参数**

在路由文件src/router/index.js里配置name属性

```js
routes: [
    {
      path: '/',
      name: 'Hello',
      component: Hello
    }
]
```

模板里(src/App.vue)用`$router.name`来接收

比如：`<p>{{ $router.name}}</p>`

**②通过`<router-link>` 标签中的to传参**

这种传参方法的基本语法：

```vue
<router-link :to="{name:xxx,params:{key:value}}">valueString</router-link>
```

比如先在src/App.vue文件中

```vue
<router-link :to="{name:'hi1',params:{username:'jspang'，id:'555'}}">Hi页面1</router-link>
```

然后把src/router/index.js文件里给hi1配置的路由起个name,就叫hi1.

```js
{path:'/hi1',name:'hi1',component:Hi1}
```

最后在模板里(src/components/Hi1.vue)用`$route.params.username`进行接收.

```vue
<template>{{$route.params.username}}-{{$route.params.id}}</template>
```

**③vue-router 利用url传递参数**----在配置文件里以冒号的形式设置参数。

我们在/src/router/index.js文件里配置路由

```js
{
    path:'/params/:newsId/:newsTitle',
    component:Params
}
```

我们需要传递参数是新闻ID（newsId）和新闻标题（newsTitle）.所以我们在路由配置文件里制定了这两个值。 在src/components目录下建立我们params.vue组件，也可以说是页面。我们在页面里输出了url传递的的新闻ID和新闻标题。

```vue
<template>
    <div>
        <h2>{{ msg }}</h2>
        <p>新闻ID：{{ $route.params.newsId}}</p>
        <p>新闻标题：{{ $route.params.newsTitle}}</p>
    </div>
</template>
<script>
export default {
  name: 'params',
  data () {
    return {
      msg: 'params page'
    }
  }
}
</script>
```

在App.vue文件里加入我们的`<router-view>`标签。这时候我们可以直接利用url传值了

`<router-link to="/params/198/jspang website is very good">params</router-link>`|

### 单页面多路由区域操作

在一个页面里我们有2个以上`<router-view>`区域，我们通过配置路由的js文件，来操作这些区域的内容

①App.vue文件，在`<router-view>`下面新写了两行`<router-view>`标签,并加入了些CSS样式

```vue
<template>
  <div id="app">
    <img src="./assets/logo.png">
       <router-link :to="{name:'HelloWorld'}"><h1>H1</h1></router-link>
       <router-link :to="{name:'H1'}"><h1>H2</h1></router-link>
    <router-view></router-view>
    <router-view name="left" style="float:left;width:50%;background-color:#ccc;height:300px;"/>
    <router-view name="right" style="float:right;width:50%;background-color:yellowgreen;height:300px;"/>
  </div>
</template>
```

②需要在路由里配置这三个区域，配置主要是在components字段里进行

```js
export default new Router({
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      components: {
        default: HelloWorld,
        left:H1,//显示H1组件内容'I am H1 page,Welcome to H1'
        right:H2//显示H2组件内容'I am H2 page,Welcome to H2'
      }
    },
    {
      path: '/h1',
      name: 'H1',
      components: {
        default: HelloWorld,
        left:H2,//显示H2组件内容
        right:H1//显示H1组件内容
      }
    }
  ]
})
```

![img](/images/vue/vue-router-demo2.png)

### vue-router配置子路由(二级路由)

实际生活中的应用界面，通常由多层嵌套的组件组合而成。同样地，URL中各段动态路径也按某种结构对应嵌套的各层组件，例如：

![img](/images/vue/vue-router-demo3.png)

**如何实现下图效果(H1页面和H2页面嵌套在主页中)**？

![img](/images/vue/vue-router-demo1.png)

①首先用<router-link>标签增加了两个新的导航链接

```vue
<router-link :to="{name:'HelloWorld'}">主页</router-link>
<router-link :to="{name:'H1'}">H1页面</router-link>
<router-link :to="{name:'H2'}">H2页面</router-link>
```

②在HelloWorld.vue加入`<router-view>`标签，给子模板提供插入位置

```vue
<template>
 <div class="hello">
   <h1>{{ msg }}</h1>
   <router-view></router-view>
 </div>
</template>
```

③在components目录下新建两个组件模板 H1.vue 和 H2.vue 两者内容类似，以下是H1.vue页面内容：

```vue
<template>
 <div class="hello">
   <h1>{{ msg }}</h1>
 </div>
</template>
<script>
 export default {
   data() {
     return {
       msg: 'I am H1 page,Welcome to H1'
     }
   }
 }
</script>
```

④修改router/index.js代码，子路由的写法是在原有的路由配置下加入children字段。

```js
routes: [
   {
     path: '/',
     name: 'HelloWorld',
     component: HelloWorld,
     children: [
       {path: '/h1', name: 'H1', component: H1},
       {path: '/h2', name: 'H2', component: H2}
     ]
   }
]
```

### vue-router跳转方法

```vue
<button @click="goToMenu" class="btn btn-success">Let's order！</button>
.....
<script>
  export default{
    methods:{
      goToMenu(){
        this.$router.go(-1)//跳转到上一次浏览的页面
        this.$router.replace('/menu')//指定跳转的地址
        this.$router.replace({name:'menuLink'})// 指定跳转路由的名字下
        this.$router.push('/menu')通过push进行跳转
        this.$router.push({name:'menuLink'})通过push进行跳转路由的名字下
      }
    }
  }
</script>
```

### 404页面的设置

用户会经常输错页面，当用户输错页面时，我们希望给他一个友好的提示页面，这个页面就是我们常说的404页面。vue-router也为我们提供了这样的机制。 ①设置我们的路由配置文件（/src/router/index.js）

```js
{
   path:'*',
   component:Error
}
```

这里的path:'*'就是输入地址不匹配时，自动显示出Error.vue的文件内容

②在/src/components/文件夹下新建一个Error.vue的文件。简单输入一些有关错误页面的内容。

```vue
<template>
    <div>
        <h2>{{ msg }}</h2>
    </div>
</template>
<script>
export default {
  data () {
    return {
      msg: 'Error:404'
    }
  }
}
</script>
```

此时我们随意输入一个错误的地址时，便会自动跳转到404页面
