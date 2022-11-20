---
title: 有useMemo,为什么还要有useCallback
date: 2022-09-20 23:57
tags: react
---

一般Js上创建一个函数需要的时间并不至于要缓存的程度，那为什么要专门给缓存函数的创建做一个语法糖呢?
**这就和React.memo有关系了。**

因为React.memo的默认第二参数是浅对比(shallow compare)上次渲染的props和这次渲染的props，如果你的组件的props中包含一个回调函数，并且这个函数是在父组件渲染的过程中创建的(见下例)，那么每次父组件(下例中的MyComponent组件）渲染时，React是认为你的子组件(下例中的Button组件)props是有变化的，不管你是否对这个子组件用了React.memo，都无法阻止重复渲染。这时就只能用useCallback来缓存这个回调函数，才会让React(或者说Js)认为这个prop和上次是相同的。

```jsx
// 下面三种方法都会在MyComponent渲染的过程中重新创建这个回调函数
// 这样都会引起Button的重新渲染 因为Button的props变化了
function MyComponent() {
  return <Button onClick={() => doWhatever()} />;
}

function MyComponent() {
  const handleClick = () => doWhatever();
  return <Button onClick={handleClick} />;
}

function MyComponent() {
  function handleClick(){ 
    doWhatever();
  }
  return <Button onClick={handleClick} />;
}

// 只有使用useCallback， 才会导致即使MyComponent渲染，也不重新创建一个新的回调函数
// 这样就不会引发Button的重新渲染 因为Button的props没变
function MyComponent() {
  const handleClick = React.useCallBack(() => doWhatever(), []);
  return <Button onClick={handleClick} />;
}
```