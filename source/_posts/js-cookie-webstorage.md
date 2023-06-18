---
title: 【JS】Cookie 和 WebStorage 的区别
date: 2023-03-26 13:59:27
categories: JavaScript
tags:
- javascript
- cookie
- local-storage
- session-storage
---

Cookie是在HTML4中使用的给客户端保存数据的，也可以和Session配合实现跟踪浏览器用户身份；

WebStorage（LocalStorage/SessionStorage）是在HTML5提出来的，纯粹为了保存数据，不会与服务器端通信。

WebStorage两个主要目标：

- 提供一种在Cookie之外存储会话数据的路径
- 提供一种存储大量可以跨会话存在的数据的机制

<!-- more -->

### 相同点

- Cookie、LocalStorage，SessionStorage都是在客户端保存数据的
- 存储数据的类型：都是字符串

### 区别

- 生命周期
  - Cookie如果不设置有效期，那么就是临时存储（存储在内存中），是会话级别的，会话结束后，Cookie也就失效了，如果设置了有效期，那么Cookie存储在硬盘里，有效期到了，就自动消失了
  - LocalStorage的生命周期是永久的，关闭页面或浏览器之后LocalStorage中的数据也不会消失。LocalStorage除非主动删除数据，否则数据永远不会消失
  - SessionStorage仅在当前会话下有效。sessionStorage引入了一个“浏览器窗口”的概念，SessionStorage是在同源的窗口中始终存在的数据。只要这个浏览器窗口没有关闭，即使刷新页面或者进入同源另一个页面，数据依然存在。但是SessionStorage在关闭了浏览器窗口后就会被销毁。同时独立的打开同一个窗口同一个页面，SessionStorage也是不一样的

- 网络流量

  Cookie的数据每次都会发给服务器端，而Localstorage和SessionStorage不会与服务器端通信，纯粹为了保存数据，所以Webstorage更加节约网络流量

- 大小限制

  Cookie大小限制在4KB，非常小；Localstorage和SessionStorage在5M

- 安全性

  WebStorage不会随着HTTP header发送到服务器端，所以安全性相对于Cookie来说比较高一些，不会担心截获

- 易用性

  WebStorage提供了一些方法，数据操作比Cookie方便

  - setItem(key, value)  保存数据，以键值对的方式储存信息
  - getItem(key)  获取数据，将键值传入，即可获取到对应的value值
  - removeItem(key)  删除单个数据，根据键值移除对应的信息
  - clear()  删除所有的数据
  - key(index)  获取某个索引的key

  

### 问题扩展

#### Cookie和Session的区别

session是存储服务器端，cookie是存储在客户端，所以session的安全性比cookie高。获取session里的信息是通过存放在会话cookie里的session id获取的。而session是存放在服务器的内存中里，所以session里的数据不断增加会造成服务器的负担，所以会把很重要的信息存储在session中，而把一些次要东西存储在客户端的cookie里。
session的信息是通过sessionid获取的，而sessionid是存放在会话cookie当中的，当浏览器关闭的时候会话cookie消失，所以sessionid也就消失了，但是session的信息还存在服务器端。

