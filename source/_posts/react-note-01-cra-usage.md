---
title: 【React】create-react-app 基础运用
date: 2023-06-18 19:54:05
categories:
- React
tags:
- react
- create-react-app
---

使用React脚手架工具 `create-react-app` 搭建项目

<!-- more -->


## 安装脚手架

```bash
# 安装脚手架
$ sudo npm install create-react-app -g

# 检查安装情况
$ create-react-app --version
# output: 5.0.1
```

## 基于脚手架创建React工程化项目

```bash
$ create-react-app [project-name]
# 项目名称要遵循npm包命名规范：使用“数字、小写字母、_“命名
```

### 项目目录

```
.
├── node_modules        # 依赖模块
├── public              # 静态资源
│   ├── favicon.ico     # 浏览器图标
│   └── index.html      # 页面模板
├── src                 # 源代码 「打包的时候一般只针对src目录进行处理」
│   └── index.js        # 入口文件
└── package.json        # 项目配置
```

#### 项目配置

`./package.json`

```json
{
  "name": "react-cra-demo",
  "version": "0.1.0",
  "private": true,
  "dependencies": { // 项目依赖
    "@testing-library/jest-dom": "^5.16.5",
    "@testing-library/react": "^13.4.0",
    "@testing-library/user-event": "^13.5.0",
    "react": "^18.2.0", // React框架的核心
    "react-dom": "^18.2.0", // React视图渲染的核心
    "react-scripts": "5.0.1", // 封装/重写了webpack配置
    "web-vitals": "^2.1.4" // 性能检测工具
  },
  "scripts": { // 命令脚本（打包命令是基于react-scripts处理）
    "start": "react-scripts start", // 开发环境：在本地启动web服务，预览打包内容
    "build": "react-scripts build", // 生产环境：打包部署，打包内容输入到build(dist)目录中
    "test": "react-scripts test", // 单元测试
    "eject": "react-scripts eject" // 暴露webpack配置规则（修改默认配置）
  },
  /**
   * 对webpack中ESLint词法检测的相关配置
   * + 词法错误「不符合标准规范」
   * + 符合标准，代码本身不会报错，但是不符合ESLint的检测规范
  */
  "eslintConfig": { 
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  /**
   * 基于browserslist规范，设置浏览器的兼容情况
   * 1. postcss-loader + autoprefixer 会给CSS3设置相关的前缀
   * 2. babel-loader 会把ES6编译为ES5
   * ...
   */
  "browserslist": {
    "production": [
      ">0.2%", // 使用率超过2%的浏览器
      "not dead", //没有死亡的浏览器 => 不考虑IE
      "not op_mini all" // 不考虑欧朋浏览器
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
    /**
     * 默认不兼容低版本和IE浏览器
     * 配置兼容IE：(IE8以上)
     *   >0.2%
     *   last 2 versions
     *   not ie <= 8
     */
  }
}
```

一个React项目中，默认会安装

- react：React框架的核心
- react-dom：React视图渲染的核心「基于React构建WebApp（HTML页面）」
- react-scripts：把 `webpack` 打包的规则和相关 `plugin` / `loader` 等都隐藏到了 `node_modules` 目录下，`react-script` 就是脚手架对打包命令的封装， 基于它打包，会调用 `node_modules` 下的 `webpack` 等进行处理

#### 入口文件

`./src/index.js`

```react
import React from 'react';
import ReactDOM from 'react-dom/client';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<div>Hello, World!</div>);
```

#### 页面模板

`./public/index.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```


## Code

https://github.com/edwhpt/react-cra-template

