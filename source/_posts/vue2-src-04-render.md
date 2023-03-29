---
title: 【Vue源码学习】4.模版渲染（render）
date: 2023-03-29 12:58:04
categories:
- Vue
- Vue2-Source
tags: 
- Vue
- Vue2-Source
---

Vue的核心流程：

1. 创造了响应式数据
2. 将模板转换成ast语法树
3. 将 `AST` 语法树转换成了 `render` 函数
4. 后续每次数据更新可以只执行 `render` 函数（无需再次执行 `AST` 转化的过程）

`render` 函数会产生 `虚拟DOM`（使用响应式数据）
根据生成的 `虚拟DOM` 创造 `真实DOM`

<!-- more -->

### mountComponent 

通过 `mountComponent` 方法实现组件的挂载

```js
Vue.prototype.$mount = function (el) {
    const vm = this
    el = document.querySelector(el)
		// 获取render ...
    mountComponent(vm, el) // 挂载组件
}
```

- 调用 `render` 方法，创建 `虚拟DOM`
- 调用 `update` 方法，根据 `虚拟DOM` 产生 `真实DOM` ，将 `真实DOM` 插入到 `el` 元素中

```js
export function mountComponent(vm, el) {
  vm.$el = el // 把el挂载到Vue实例上，这里的el是通过querySelector处理过的 (#app)
  const vnode = vm._render() // vm.$options.render() => 虚拟DOM
  vm._update(vnode) // 根据虚拟DOM产生真实DOM，将真实DOM插入到el元素中
}
```

扩展生命周期方法，在Vue原型上添加 `render`和`update`

```js
function Vue(options) {
  this._init(options) // options就是用户的选项
}

initMixin(Vue)
initLifeCycle(Vue)

export default Vue
```

```js
export function initLifeCycle(Vue) {
    Vue.prototype._render = function () {}
  	Vue.prototype._update = function () {}
}
```

### render

在Vue原型上扩展 `_render` 方法

```js
Vue.prototype._render = function () {
    // 当渲染的时候会去实例中取值，我们就可以将属性和视图绑定在一起
    const vm = this
    vm.$options.render.call(vm) // 通过AST语法转义后生成的render方法
}
```

在之前的 Vue $mount过程中，我们已通过 compileToFunction方法将模版template 编译成 render方法，其返回一个 虚拟DOM。template转化成render函数的结果如下

```html
<!-- template -->
<div id="app" style="color: lightgreen;background: black;">
  <h1>hello, {{name}}</h1>
  <span></span>
</div>
```

```js
// render
ƒ anonymous() {
	with(this){
    return _c('div', {id:"app",style:{"color":" lightgreen","background":" black"}},
      _c('h1',null,_v("hello,"+_s(name))),
      _c('span', null))
  }
}
```

render 内部使用了 `_c`、`_v` 、`_s` 方法，需要在Vue原型扩展这些方法

- _c 创建虚拟DOM标签
- _v 创建虚拟DOM文本
- _s 处理变量

```js
// _c("div", {}, ...children)
Vue.prototype._c = function () {
    return createElementVNode(this, ...arguments)
}
// _v(text)
Vue.prototype._v = function () {
    return createTextVNode(this, ...arguments)
}

Vue.prototype._s = function (value) {
    if (typeof value !== 'object') return value
    return JSON.stringify(value)
}
```

#### vdom

处理标签和文本，返回 `VNode`

```js
// h()  _c() 创建虚拟DOM标签
export function createElementVNode(vm, tag, data = {}, ...children) {
    if (data == null) {
        data = {}
    }
    let key = data.key
    if (key) {
        delete data.key
    }
    return vnode(vm, tag, key, data, children)
}

// _v() 创建虚拟DOM文本
export function createTextVNode(vm, text) {
    return vnode(vm, undefined, undefined, undefined, undefined, text)
}

function vnode(vm, tag, key, data, children, text) {
    return { vm, tag, key, data, children, text }
}
```

> AST和VNode的区别
> AST：是语法层面的转化，描述的是语法本身（可以描述JS、CSS ...）
> 虚拟DOM：是描述的DOM元素，可以增加一些自定义属性（描述DOM的）

### update

`update` 内部通过调用 `patch` 方法，将 `render` 返回的 `VNode` 转成 `真实DOM`

```js
Vue.prototype._update = function (vnode) {
  const vm = this
  const el = vm.$el
  vm.$el = patch(el, vnode) // 完成渲染，返回更新后的DOM
}
```

#### patch

`patch` 既有初始化元素的功能 ，又有更新元素的逻辑；我们先不管更新的部分

首次渲染流程：

- 通过 `createElm` 方法递归创建 `DOM` 树
- 在页面插入新创建的 `DOM` 树
- 删除页面上原有的 `DOM` 节点

```js
function patch(oldVNode, vnode) {
    const isRealElement = oldVNode.nodeType
    if (isRealElement) {
        // 首次渲染流程
        const elm = oldVNode // 获取真实DOM元素
        const parentElm = elm.parentNode // 获取父元素
        let newElm = createElm(vnode) // 获取新节点
        parentElm.insertBefore(newElm, elm.nextSibling) // 插入新节点
        parentElm.removeChild(elm) // 删除老节点
        return newElm
    } else {
        // 更新流程
        // Todo: diff算法
    }
}

function patchProps(el, props) {
    for (const key in props) {
        // 处理样式
        if (key === 'style') {
            for (let styleName in props.style) {
                el.style[styleName] = props.style[styleName]
            }
        } else {
            el.setAttribute(key, props[key])
        }
        // Todo: 处理其他属性 ...
    }
}

function createElm(vnode) {
    let { tag, data, children, text } = vnode

    if (typeof tag === 'string') {
        // 处理标签
        // 将真实节点和虚拟节点对应起来，后续如果修改属性可以通过虚拟节点找到真实节点，然后完成更新
        vnode.el = document.createElement(tag)

        // 处理属性
        patchProps(vnode.el, data)

        // 处理子元素，递归创建子节点
        children.forEach((child) => {
            vnode.el.appendChild(createElm(child))
        })
    } else {
        // 处理文本
        vnode.el = document.createTextNode(text)
    }

    return vnode.el
}
```

