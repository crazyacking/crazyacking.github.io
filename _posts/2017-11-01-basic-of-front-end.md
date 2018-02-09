---
title: 初探前端
description: 如果说html是一个简单的静态网页，就像人素颜一样。css样式会让页面更加酷炫，就像化上妆一样，js让页面可以动态变化，给页面添加了动画效果，检测访客的浏览信息。
categories:
 - front-end
tags: front-end, weex, js
---


## Weex

Weex是基于Vue.js的。

Weex 表面上是一个客户端技术，但实际上它串联起了从本地开发环境到云端部署和分发的整个链路。

**参考文档**

[weex-project.io](https://weex-project.io/cn/guide/intro/how-it-works.html)

### 原理

开发者首先可以在本地像撰写 web 页面一样撰写一个 app 的页面，然后编译成一段 JavaScript 代码，形成 Weex 的一个 JS bundle；

在云端，开发者可以把生成的 JS bundle 部署上去，然后通过网络请求或预下发的方式传递到用户的移动应用客户端；

在移动应用客户端里，WeexSDK 会准备好一个 JavaScript 引擎，并且在用户打开一个 Weex 页面时执行相应的 JS bundle，并在执行过程中产生各种命令发送到 native 端进行的界面渲染或数据存储、网络通信、调用设备功能、用户交互响应等移动应用的场景实践

同时，如果用户没有安装移动应用，他仍然可以在浏览器里打开一个相同的 web 页面，这个页面是使用相同的页面源代码，通过浏览器里的 JavaScript 引擎运行起来的。
![img](/assets/images/posts/2017-11-01-basic-of-front-end-1.png)

Weex 的本地开发环境基于 web 开发体验而设计，web 开发者可以通过自己熟悉的 HTML/CSS/JavaScript 技术和语法实现移动应用的界面。

如果你知道WebView，可以按照WebView的方式去理解Weex。但Weex并不是基于WebView，准确说和浏览器共通的是V8引擎而已。

## Vue.js

想必现在能看到我这篇文章的人，都是用着APP或者网页版在阅读吧。Vue.js就是一个用于搭建类似于网页版知乎这种表单项繁多、且内容需要根据用户的操作进行修改的网页版应用。

基于HTML 的模板语法，允许开发者声明式地将DOM 绑定至底层Vue 实例的数据。

**参考文档**

[Vue.js新手入门指南](https://zhuanlan.zhihu.com/p/25659025)

Vue.js为什么能让基于网页的前端应用程序开发起来这么方便？ 因为Vue.js有声明式，响应式的数据绑定，与组件化的开发，并且还使用了Virtual DOM这个看名字就觉得高大上的技术。响应式的数据绑定是指vue.js会自动对页面中某些数据的变化做出响应。

## webView

WebView是手机中内置了一款高性能 webkit 内核浏览器，在 SDK 中封装的一个组件。没有提供地址栏和导航栏，WebView只是单纯的展示一个网页界面。在开发中经常都会用到。

## JavaScript

我们常说的JS就是指JavaScript。

JS是无处不在的脚本语言，还被封装了各种框架，如react.js、angular.js、vue.js、ext.js、iMAG.js、node.js等

Angular，React，Vue，这三者其实面对的是同一个领域，那就是Web应用。

这三者中，Angular的适用领域相对窄一些，React可以拓展到服务端、移动端Native部分，而Vue因为比较轻量，还能用于业务场景非常轻的页面中。

其中，node.js又被称为后端js，和传统前端js相比，后端js需要掌握更多后端技术（比如web服务器原理等）。