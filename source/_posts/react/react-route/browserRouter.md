---
title: BrowserRouter
date: 2022-01-07 14:04
tags: router 
---

BrowserRouter 使用HTML5的history API（pushState, replaceState和popState），让页面的UI与URL同步

HashRouter使用的是URL的hash部分（即window.location.hash），来保持页面的UI与URL的同步。永远不会被发送到服务器（not recommend）

routes 将URL与组件、数据加载和数据突变耦合在一起。通过路由嵌套，复杂的应用程序布局和数据依赖关系变得简单明了。