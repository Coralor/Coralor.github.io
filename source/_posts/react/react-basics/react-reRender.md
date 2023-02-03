---
title: react 的re-render
date: 2022-12-13 00:36
tags: react
---

react组件的props和state发生改变时，会触发组件的重新渲染。

# setState的执行原理
1. setState接收new State
2. 接收到的状态不会立即执行，而是存入pendingState(等待队列)
3. 判断isBatchingUpdates(是否是批量更新)
   1) isBatchingUpdate: true, 将new State 存入到dirtyComponents
   2) isBatchingUpdate: false, 遍历所有的dirtyComponents, 并且调用updateComponent方法更新pendingStates中的state或props。执行完成后，isBatchingUpdates:true

``` jsx
  import React, { useState, useEffect, useRef } from "react";

export default function excersise() {
  const [params1, setParams1] = useState<number>(0);
  const containerRef = useRef<HTMLParagraphElement>(null);

  const timerCallBack = () => {
    setTimeout(() => {
      setParams1((pre) => pre + 1);
      setParams1((pre) => pre + 1);
      setParams1((pre) => pre + 1);
      // 会渲染三次， params1 = 3
    });
  };

  const asyncCallback = () => {
    Promise.resolve().then(() => {
      setParams1((pre) => pre + 1);
      setParams1((pre) => pre + 1);
      setParams1((pre) => pre + 1);
      // 会渲染三次， params1 = 3
    });
  };

  const reactEventCallback = () => {
    setParams1((pre) => pre + 1);
    setParams1((pre) => pre + 1);
    setParams1((pre) => pre + 1);
    // 只会渲染一次， params1 = 1
  };

  useEffect(() => {
    const handler = () => {
      setParams1((pre) => pre + 1);
      setParams1((pre) => pre + 1);
      setParams1((pre) => pre + 1);
      // 会渲染三次， params1 = 3
    };
    containerRef.current?.addEventListener("click", handler);
    return () => containerRef.current?.removeEventListener("click", handler);
  }, []);

  const updateUnuse = () => {
    setParams4((pre) => pre + 1);
    // 会渲染一次
  };

  return (
    <div>
      <p onClick={timerCallBack}>定时器回调 </p>
      <p onClick={asyncCallback}>异步函数回调</p>
      <p onClick={reactEventCallback}>React合成事件</p>
      <p ref={containerRef}>原生事件</p>
      <p onClick={updateUnuse}>更新未在页面使用的state</p>
      <p>参数1变化：{params1} </p>
    </div>
  );
}

```

**1. 在定时器回调，异步函数回调，原生事件中状态的改变都是同步的，并不会被合并。因此它们在变化的过程中都re-render了三次。**
**2. 而在react的合成事件，多次的setState都会被合并成一次更新，所以只会re-render一次**

因此setState无所谓同步还是异步，只是看是否命中batchUpdate机制，判断isBatchingUpdates。

## 那么哪些可以命中batchUpdate机制？
+ 生命周期（和它调用的函数）
+ React中注册的事件（和它调用的函数）
+ React可管理的入口
  
## 那么哪些不能命中batchUpdate机制？
+ setTimeout, setInterval等（和他调用的函数）
+ 自定义的dom事件（和它调用的函数）
+ React管理不了的入口（异步回调等）

总而言之，
+ setState只有在React合成事件和钩子函数中是"异步"的，在定时器，异步回调，原生事件中都是同步的，不触发批量更新。
+ setState的"异步"并非是说内部是由异步代码实现，本身执行过程和代码都是同步的，只是合成事件和钩子函数的调用顺序实在更新之前，导致在合成事件和更新函数中没法立马拿到更新后的值，形成了所谓的"异步"。
+ setStatede的批量更新优化是建立在"异步"之上。**在“异步”中，如果对同一个值进行多次setState,setState的批量更新策略会对其进行覆盖，取最后一次执行。如果是同时setState多个不同的值那么在更新时会对其进行合并批量更新**。