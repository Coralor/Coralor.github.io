---
title: react-router-dom v6
date: 2022-10-23 21:56
tags: react
---
React Route V6

# React Router提供了两种方法以供我们声明路由：

+ Routes 和 Route
+ useRoutes

# 路由跳转
+ Link 和 NavLink 渲染一个<a> 元素
+ useNavigate 和 Navigate 跳转导航，通常用在事件中，或者响应状态变化。

# Routes和Route
+ <Routes> 和 <Route> 是React Router中基于当前location渲染页面的主要方法。你可以把Route当成if条件，如果path 匹配当前的URL，则渲染它的element，否则不渲染。<Route caseSensitive>判断大小写是否敏感，默认是false。

+ 当location改变的时候， <Routes>遍历它所有的子Route，然后渲染匹配的Route。 <Route>可能是嵌套的，嵌套路由也对应URL。父路由通过渲染 <Outlet>来渲染子路由。

```tsx 
    <Routes>
        <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="product" element={<Product />} />
        </Route>
    </Routes>
```
默认的 <Route element> 是一个 Outlet，这意味着即使没有明确的 element，路由也可以渲染它的子节点。因此我们可以在没有嵌套UI的情况下嵌套路径。

```tsx 
    <Route path="users">
      <Route path=":id" element={<UserProfile />} />
    </Route>
```

# Outlet

<Outlet>用在父路由中，这样在渲染子路由的时候，内部嵌套的UI也会被渲染。如果父路由是精确匹配， <Outlet>则会渲染一个标记index的子路由，如果没有index的子路由，那就什么都不渲染

# 404 path设置为*

```tsx
<Route path="*" element={<NoMatch />} />
```

