---
title: 【Vue2源码学习】1.使用Rollup搭建开发环境
date: 2023-03-27 13:27:16
categories: 
- Vue
- Vue2 Source
tags:
- vue
- rollup
- build
---

使用Rollup快速搭建开发环境
<!-- more -->

### 初始化环境

```bash
$ npm init -y
```

### 安装依赖

- rollup：打包工具，相对于webpack打包体积更小
- rollup-plugin-babel：rollup环境的babel插件，负责编译JS高级语法
- @babel/core：babel核心模块
- @babel/preset-env：babel预设插件，把高级语法转换为低级语法，比如let/const转换为var，箭头函数、类的转换

```bash
$ npm install rollup rollup-plugin-babel @babel/core @babel/preset-env --save-dev
```

### 创建打包入口文件 src/index.js

### 在根目录创建rollup配置文件，默认名字为 rollup.config.js

```js
import babel from 'rollup-plugin-babel'

// rollup默认可以导出一个对象，作为打包的配置文件
export default {
    input: './src/index.js', // 入口文件
    output: {
        file: './dist/vue.js', // 出口文件
        name: 'Vue', // 全局添加Vue属性，global.Vue
        format: 'umd', // esm es6模块  commonjs模块  iife自执行函数  umd（支持commonjs、amd）
        sourcemap: true // 希望可以调试源代码
    },
    // 配置一些插件
    plugins: [
      // 所有插件都是函数，直接执行
      babel({
        exclude: 'node_modules/**' // 排除node_modules所有文件
      })
    ]
}
```

### 创建babel配置文件 .babelrc，打包时使用babel会默认采用配置文件中的配置属性

```json
{
  "presets": [
    "@babel/preset-env"
  ]
}
```

### 在package.json中配置rollup命令脚本

- rollup -c（rollup --config）打包时使用配置文件（rollup.config.js）里面的配置属性
- rollup -w (rollup --watch）监控文件变化，文件发生变化时重新打包

```json
{
  "scripts": {
    "dev": "rollup -cw"
  }
}
```

