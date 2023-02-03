---
title: js宏任务和微任务
date: 2022-08-21 20:50:24
tags:
---

JS是单线程的语言，然而浏览器却又可以很好地处理异步请求，这是为什么呢？

## 同步任务和异步任务

JS 执行过程中会产生两种任务：同步任务和异步任务。

+ 同步任务：比如声明语句、for、赋值等，读取后依据从上到下从左到右，立即执行。
+ 异步任务：比如 ajax 网络请求，setTimeout 定时函数等都属于异步任务。异步任务会通过任务队列(Event Queue)的机制（先进先出的机制）来进行协调。
  
!['任务执行机制'](/images/task.png)

流程图解释：

JavaScript引擎在 等待任务，执行任务 和 进入休眠状态等待更多任务 这几个状态之间转换的无限循环。

+ 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入Event Table并注册函数。
+ 当指定的事情完成时，Event Table会将这个函数移入Event Queue。
+ 主线程内的任务执行完毕为空，会去Event Queue读取对应的函数，进入主线程执行。
+ 上述过程会不断重复，也就是常说的Event Loop(事件循环)。
+ **那怎么知道主线程执行栈为空啊？**
js引擎存在monitoring process进程，会持续不断的检查主线程执行栈是否为空，一旦为空，就会去Event Queue那里检查是否有等待被调用的函数。

## 宏任务和微任务

JS中的异步任务也分为两种：**宏任务（Macro-take）和微任务（Micro-take）**
**在es5后，javascript引入了Promise，这样，不需要浏览器，js引擎自身也可以发起异步任务。**

+ 宏任务是由宿主(浏览器，node)发起，微任务由js引擎发起的。
+ 宏任务主要包括：scrip(JS 整体代码)、setTimeout、setInterval、setImmediate、I/O、UI 交互，网络请求（AJAX/Fetch)
+ 微任务主要包括：Promise(重点关注, Promise本身是同步的，但是then，catch是异步)、process.nextTick(Node.js)、MutaionObserver、Async/Await、Object.observe

**任务队列的执行过程是：先执行一个宏任务，执行过程中如果产出新的宏/微任务，就将他们推入相应的任务队列，之后执行一队列微任务，之后再执行宏任务，如此循环。以上不断重复的过程就叫做 Event Loop(事件循环)。**
每一次的循环操作被称为tick。

!['任务队列的执行过程'](/images/taskProcess.png)

执行的代码可能有三种：
1. 同步代码（JS执行栈、回调栈)
2. 微任务的异步代码（JS引擎）
3. 宏任务的异步代码(宿主环境)

## 到底先执行宏任务还是微任务？

### 原则一

+ 万物皆从全局上下文准备退出，全局的同步代码运行结束的这个时机开始

```javascript
function app() {
      setTimeout(() => {
        console.log("1-1");
        Promise.resolve().then(() => {
          console.log("2-1");
        });
      });
      console.log("1-2");
      Promise.resolve().then(() => {
        console.log("1-3");
        setTimeout(() => {
          console.log("3-1");
        });
      });
    }
    app();
```

当执行完了console.log("1-2")的时候，意味着全局的上下文马上要退出了,
因为此时全局的同步代码都执行完了,剩下的都是异步代码

### 原则二

+ 同一层级下(不理解层级，可以先不管，后面会讲),微任务永远比宏任务先执行
+ 即Promise.then比setTimeout先执行
+ 所以先打印1-3,再打印1-1

### 原则三

+ **每个宏任务,都单独关联了一个微任务队列**
+ 每个层级的宏任务,都对应了他们的微任务队列,微任务队列遵循先进先出的原则,当全局同步代码执行完毕后,就开始执行第一层的任务。同层级的微任务永远先于宏任务执行,并且会在当前层级宏任务结束前全部执行完毕

#### 怎么分辨层级？

+ 属于同一个维度的代码,例如下面的func1和func2就属于同层级任务
  
```javascript
setTimeout(func1)...
Promise.resolve().then(func2)...
```

+ 下面这种fn1和fn2就不属于同一个层级的,因为fn2属于内部这个setTimeout的微任务队列,而fn1属于外部setTimeout的微任务队列
  
```javascript
setTimeout(()=>{
  Promise.resolve().then(fn1)
  setTimeout(()=>{
    Promise.resolve().then(fn2)  
  })
})
```

## 举例理解宏任务与微任务的执行过程

```javascript
console.log("script start");

setTimeout(function () {
  console.log("setTimeout");
}, 0);

Promise.resolve()
  .then(function () {
    console.log("promise1");
  })
  .then(function () {
    console.log("promise2");
  });

console.log("script end");
```

按照上面的内容，分析执行步骤：
1.宏任务：执行整体代码（相当于script中的代码）：
  a.输出: script start
  b.遇到 setTimeout，加入宏任务队列，当前宏任务队列(setTimeout)
  c.遇到 promise，加入微任务，当前微任务队列(promise1)
  d.输出：script end
2.微任务：执行微任务队列（promise1）
  a.输出：promise1，then 之后产生一个微任务，加入微任务队列，当前微任务队列（promise2）
  b.执行 then，输出promise2
  c.执行渲染操作，更新界面（敲黑板划重点）。
  d.宏任务：执行 setTimeout
  e.输出：setTimeout

### Promise 的执行

new Promise(..)中的代码，也是同步代码，会立即执行。只有then之后的代码，才是异步执行的代码，是一个微任务。

```javascript
console.log("script start");

setTimeout(function () {
  console.log("timeout1");
}, 10);

new Promise((resolve) => {
  console.log("promise1");
  resolve();
  setTimeout(() => console.log("timeout2"), 10);
}).then(function () {
  console.log("then1");
});

console.log("script end");
```

步骤解析：

+ 当前任务队列：微任务: [], 宏任务：[script]
  
宏任务：
输出: script start
遇到 timeout1，加入宏任务
遇到 Promise，输出promise1，直接 resolve，将 then 加入微任务，遇到 timeout2，加入宏任务。
输出script end
宏任务第一个执行结束

+ 当前任务队列：微任务[then1]，宏任务[timeou1, timeout2]
  
微任务：
执行 then1，输出then1
微任务队列清空

+ 当前任务队列：微任务[]，宏任务[timeou1, timeout2]
  
宏任务：
输出timeout1
输出timeout2

+ 当前任务队列：微任务[]，宏任务[timeou2]
  
微任务：
为空跳过

+ 当前任务队列：微任务[]，宏任务[timeou2]
  
宏任务：
输出timeout2

### async/await 的执行

```javascript
// async/await 写法
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
// Promise 写法
async function async1() {
  console.log("async1 start");
  Promise.resolve(async2()).then(() => console.log("async1 end"));
}
```

```javascript
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
async function async2() {
  console.log("async2");
}
async1();
setTimeout(() => {
  console.log("timeout");
}, 0);
new Promise(function (resolve) {
  console.log("promise1");
  resolve();
}).then(function () {
  console.log("promise2");
});
console.log("script end");
```

步骤解析：

+ 当前任务队列：宏任务：[script]，微任务: []
  
宏任务：
输出：async1 start
遇到 async2，输出：async2，并将 then（async1 end）加入微任务
遇到 setTimeout，加入宏任务。
遇到 Promise，输出：promise1，直接 resolve，将 then(promise2)加入微任务
输出：script end

+ 当前任务队列：微任务[promise2, async1 end]，宏任务[timeout]

微任务：
输出：promise2
promise2 出队
输出：async1 end
async1 end 出队
微任务队列清空

+ 当前任务队列：微任务[]，宏任务[timeout]
  
宏任务：
输出：timeout
timeout 出队，宏任务清空

### setTimerout 并不准确

**setTimeout() 的第二个参数是为了告诉 JavaScript 再过多长时间把当前任务添加到队列中。**
**如果队列是空的，那么添加的代码会立即执行；如果队列不是空的，那么它就要等前面的代码执行完了以后再执行。**

```javascript
const s = new Date().getSeconds();
console.log("script start");
new Promise((resolve) => {
  console.log("promise");
  resolve();
}).then(() => {
  console.log("then1");
  while (true) {
    if (new Date().getSeconds() - s >= 4) {
      console.log("while");
      break;
    }
  }
});
setTimeout(() => {
  console.log("timeout");
}, 2000);
console.log("script end");
```

**因为then是一个微任务，会先于setTimeout执行。
虽然setTimeout是在两秒后加入的宏任务，但是因为then中的在while操作被延迟了4s，
所以一直推迟到了4s秒后才执行setTimeout。

所以输出的顺序是：script start、promise、script end、then1。
四秒后输出：while、timeout**

注意：关于 setTimeout 要补充的是，即便主线程为空，0 毫秒实际上也是达不到的。根据 HTML 的标准，最低是 4 毫秒。

### process.nextTick比Promise先执行，为什么？

process.nextTick 永远大于 promise.then。在Node中，_tickCallback在每一次执行完TaskQueue中的一个任务后被调用，
而这个_tickCallback中实质上干了两件事：

1、nextTickQueue中所有任务执行掉(长度最大1e4，Node版本v6.9.1)

2、第一步执行完后执行_runMicrotasks函数，执行microtask（微任务）中的部分(promise.then注册的回调)

所以很明显 process.nextTick > promise.then

```javascript
process.nextTick(function() {
    console.log(7);
});

new Promise(function(resolve) {
    console.log(3)
    resolve();
    console.log(4);
}).then(function() {
    console.log(5)
})

process.nextTick(function() {
    console.log(8)
})
```

执行顺序： 3 4 7 8 5


<!--event loop exercise site https://www.jsv9000.app/ -->