---
title: 【Vue2源码学习】3.模板编译(complier)
date: 2023-03-28 17:57:28
categories:
- Vue
- Vue2-Source
tags:
- Vue
- Vue2-Source
---

将 `data` 数据解析到 `el` 元素上，需要将 `template` 语法转换成 `render` 函数

> 实现方式：
>
> 1. 模板引擎，性能差，需要正则匹配替换；Vue1.0的时候没有引入虚拟DOM的概念
>
> 2. 采用虚拟DOM，数据变化后比较虚拟DOM的差异，最后更新需要更新的地方
>
>    核心就是将template模板变成JS语法，通过JS语法生成虚拟DOM

<!-- more -->

## 初始化 `$mount` 方法

```js
Vue.prototype.$mount = function(el) {
  const vm = this
  el = document.querySelector(el)
  let ops = vm.$options
  
  if (!ops.render) { // 先找render函数
    let template // 没有render看一下是否有template，没有template就采用el外部元素
    if (!ops.template && el) { // 没写template，但是写了el
      if (!ops.template && el) {
        template = el.outerHTML
      } else {
        if (el) {
          template = ops.template // 如果有el，则采用模板的内容
        }
      }
    }
    
    if (template) {
      // 这里需要对模板进行编译
      const render = complieToFunction(template)
      ops.render
    }
  }
  
  ops.render // 最终就可以获取render方法
}
```

> script 标签引用的vue.global.js 这个编译过程是在浏览器运行的
>
> runtime 是不包含模板编译的，整个编译过程是打包的时候通过loader来转义.vue文件的，用runtime时不能使用template

## 通过 `complieToFunction` 方法对模版进行编译处理

- 将 `template` 转化成 `AST` 语法树
- 生成 `render` 方法（render方法返回的结果就是 `虚拟DOM`）

```js
export function compileToFunction(template) {
    // 1.将template 转化成ast语法树
    let ast = parseHTML(template)

    // 2.生成render方法 （render方法执行后返回的结果就是 虚拟DOM）
    let code = codegen(ast)
    code = `with(this){return ${code}}`
    let render = new Function(code)
    
    return render
}
```

### parse

在Vue的 `$mount` 过程中，编译过程首先就是调用 `parseHTML` 方法，解析 `template` 模版，生成 `AST` 语法树 在这个过程，我们会用到正则表达式对字符串解析，匹配开始标签、文本内容和闭合标签等

```js
const ncname = `[a-zA-Z_][\\-\\.0-9_a-zA-Z]*`
const qnameCapture = `((?:${ncname}\\:)?${ncname})`
const startTagOpen = new RegExp(`^<${qnameCapture}`) // 匹配到的分组是一个 标签名 <div 匹配到的是开始标签的名字
const endTag = new RegExp(`^<\\/${qnameCapture}[^>]*>`) // 匹配的是 </xxx> 最终匹配到的分组是结束标签的名字
const attribute = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/ // 匹配属性，第一个分组是属性的key，value可能是分组3/分组4/分组5
const startTagClose = /^\s*(\/?)>/ // <div>  <br/>
```

使用 `while` 遍历 `HTML` 字符串，利用正则去匹配开始标签、文本内容和闭合标签，然后执行 `advance` 方法将匹配到的内容在原HTML字符串中剔除，直到 `HTML` 字符串为空，结束循环

```js
export function parseHTML(html) {
  // 循环html字符串，直到其为空停止
  while (html) {
    // 如果textEnd = 0 说明是一个开始标签或者结束标签
    // 如果textEnd > 0 说明就是文本的结束位置
    let textEnd = html.indexOf('<')
    if (textEnd == 0) {
      // 开始标签的解析結果，包括 标签名 和 属性
      const startTagMatch = parseStartTag()

      if (startTagMatch) {
        start(startTagMatch.tagName, startTagMatch.attrs)
        continue
      }

      // 匹配结束标签
      let endTagMatch = html.match(endTag)
      if (endTagMatch) {
        advance(endTagMatch[0].length)
        end(endTagMatch[1])
        continue
      }
    }
    if (textEnd > 0) {
      let text = html.substring(0, textEnd) // 截取文本内容
      if (text) {
        chars(text)
        advance(text.length)
      }
    }
  }

  // 解析开始标签
  function parseStartTag() {
    const start = html.match(startTagOpen)
    if (start) {
      const match = {
        tagName: start[1], // 标签名
        attrs: [],
      }
      advance(start[0].length)

      let attr, end
      // 如果不是开始标签的结束 就一直匹配下去
      while (!(end = html.match(startTagClose)) && (attr = html.match(attribute))) {
        advance(attr[0].length)
        match.attrs.push({ name: attr[1], value: attr[3] || attr[4] || attr[5] || true })
      }

      // 如果不是开始标签的结束
      if (end) {
        advance(end[0].length)
      }
      return match
    }
    return false
  }
  
  // 剔除 template 已匹配的内容
  function advance(n) {
    html = html.substring(n)
  }
	
  // todo
  // 处理开始标签，利用栈型结构来构造一颗树
  function start(tag, attrs) {}
	// 处理文本
  function chars(text) {}
  // 处理结束标签
  function end(tag) {}
}
```

>  [htmlparse2](https://www.npmjs.com/package/htmlparser2) 实现了同样的效果，这里通过手动实现理解相关原理

当我们使用正则匹配到开始标签、文本内容和闭合标签时，分别执行 `start`、`chars`、`end`方法去处理，利用 `stack` 栈型数据结构，最终构造一颗 `AST` 语法树，即 `root`

1. 匹配到开始标签时，就创建一个 `AST` 元素，判断如果有 `currentParent`，会把当前 `AST` 元素 push 到 `currentParent.chilldren` 中，同时把 ast元素的 parent 指向 currentParent，ast元素入栈并更新 currentParent
2. 匹配到文本时，就给 currentParent.children push一个文本 ast元素
3. 匹配到结束标签时，就弹出栈中最后一个 `AST` 元素，更新 `currentParent`

currentParent：指向的是栈中的最后一个 `AST` 节点

> 注意：栈中的当前ast节点永远是下一个AST节点的父节点

```js
export function parseHTML(html) {const ELEMENT_TYPE = 1 // 元素类型
  const TEXT_TYPE = 3 // 文本类型
  const stack = [] // 用于存放元素的栈
  let currentParent // 指向的是栈中的最后一个
  let root
  
  // while ...
  // parseStartTag ...
  // advance ...

  // 最终需要转化成一颗抽象语法树
  function createASTElement(tag, attrs) {
    return {
      tag, // 标签名
      type: ELEMENT_TYPE, // 类型
      attrs, // 属性
      parent: null,
      children: [],
    }
  }

  // 处理开始标签，利用栈型结构 来构造一颗树
  function start(tag, attrs) {
    let node = createASTElement(tag, attrs) // 创造一个 ast节点
    if (!root) {
      root = node // 如果root为空，则当前是树的根节点
    }
    if (currentParent) {
      node.parent = currentParent // 只赋予了parent属性
      currentParent.children.push(node) // 还需要让父亲记住自己
    }
    stack.push(node)
    currentParent = node // currentParent为栈中的最后一个
  }

  // 处理文本
  function chars(text) {
    text = text.replace(/\s/g, '')
    // 文本直接放到当前指向的节点中
    if (text) {
      currentParent.children.push({
        type: TEXT_TYPE,
        text,
        parent: currentParent,
      })
    }
  }

  // 处理结束标签
  function end(tag) {
    stack.pop() // 弹出栈中最后一个ast节点
    currentParent = stack[stack.length - 1]
  }
  
  return root
}
```

> 可以利用AST的可视化工具网站 - [AST Exploer](https://link.juejin.cn/?target=https%3A%2F%2Fastexplorer.net%2F) ，使用各种parse对代码进行AST转换

### codegen

编译的最后一步就是把优化后的 AST树转换成可执行的 render代码。此过程包含两部分，第一部分是使用 codegen方法生成 render代码字符串，第二部分是利用模板引擎转换成可执行的 render代码 render方法代码字符串格式如下

```
`_c('div', {id:"app",style:{"color":" red","background":" white"}},_v("hello,"+_s(name)))`
```

我们会在Vue原型上扩展这些方法

> _c: 执行 createElement创建虚拟节点；
>
> _v: 执行 createTextVNode创建文本虚拟节点；
>
> _s: 处理变量；

实现一个简单的 `codegen` 方法，深度遍历 `AST` 树去生成 `render` 代码字符串

```js
function codegen(ast) {
  let children = genChildren(ast.children)
  let code = `_c('${ast.tag}',${ast.attrs.length > 0 ? genProps(ast.attrs) : 'null'}${ast.children.length ? `,${children}` : ''})`
  return code
}

// 根据ast语法树的 children对象 生成相对应的 children字符串
function genChildren(children) {
  return children.map(child => gen(child)).join(',')
}

const defaultTagRE = /\{\{((?:.|\r?\n)+?)\}\}/g // 匹配到的内容就是我们表达式的变量，例如 {{ name }}
function gen(node) {
  if (node.type === 1) {  // 元素
    return codegen(node)
  } else {  // 文本
    let text = node.text
    if (!defaultTagRE.test(text)) {
      // _v('hello')
      return `_v(${JSON.stringify(text)})`
    } else {
      //_v( _s(name) + 'hello' + _s(age))
      ... 拼接 _s
      return `_v(${tokens.join('+')})`
    }
  }
}

// 根据ast语法树的 attrs属性对象 生成相对应的属性字符串
function genProps(attrs) {
  let str = ''
  for (let i = 0; i < attrs.length; i++) {
    let attr = attrs[i]
    str += `${attr.name}:${JSON.stringify(attr.value)},` // id:'app',class:'app-inner',
  }
  return `{${str.slice(0, -1)}}`
}
```

模板引擎的实现原理就是 `with` + `new Function()`，转换成可执行的函数，最终赋值给`vm.options.render`

```js
let code = codegen(ast)
code = `with(this){return ${code}}`
let render = new Function(code) 
```

尤雨溪老师解读：[如何看待Vue.js 2.0 的模板编译使用了`with(this)`的语法?](https://www.zhihu.com/question/49929356/answer/118534768)

> 为啥呢，因为没有什么太明显的坏处（经测试性能影响几乎可以忽略），但是 with 的作用域和模板的作用域正好契合，可以极大地简化模板编译过程。Vue 1.x 使用的正则替换 identifier path 是一个本质上 unsound 的方案，不能涵盖所有的 edge case；而走正经的 parse 到 AST 的路线会使得编译器代码量爆炸。虽然 Vue 2 的编译器是可以分离的，但凡是可能跑在浏览器里的部分，还是要考虑到尺寸问题。用 with 代码量可以很少，而且把作用域的处理交给 js 引擎来做也更可靠。
>
> 用 with 的主要副作用是生成的代码不能在 strict mode / ES module 中运行，但直接在浏览器里编译的时候因为用了 new Function()，等同于 eval，不受这一点影响。
>
> 当然，最理想的情况还是可以把 with 去掉，所以在使用预编译的时候（vue-loader 或 vueify），会自动把第一遍编译生成的代码进行一次额外处理，用完整的 AST 分析来处理作用域，把 with 拿掉，顺便支持模板中的 ES2015 语法。也就是说如果用 webpack + vue 的时候，最终生成的代码是没有 with 的。

