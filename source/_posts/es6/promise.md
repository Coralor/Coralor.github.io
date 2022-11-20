---
title: Promise 的简单理解
date: 2022-10-26 22:51:24
tags: es6
---

为什么需要Promise?

Promise可以将异步操作以同步的流程表达出来，可以解决回调地狱的问题。

# Promise的含义
Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息

# Promise的基本使用
+ Promise 接受一个函数作为参数，在函数中有两个参数： reolove,reject。
+ Promise实例，有两个属性：state和result
+ Promise的状态：pending, fullfilled, rejected
  
```javascript
 new Promise((resolve, reject) => {})
```

(1) Promise状态改变
+ 调用resolve(),可以使当前Promise对象状态改变为fulfilled。**在异步操作成功是调用，并将异步操作的结果作为参数传递出去**
+ 调用reject(),可以使当前Promise对象状态变成rejected。**在异步操作失败时调用，并将异步操作报出的错误作为参数传递出去**
+ Promise 的状态一旦改变，就永久保持该状态，不会再变了
  
```javascript
  const p = new Promise((resolve, reject) => {
    resolve();    
    // reject()
  })
  console.log(p)
```
!['Promise'](/images/promiseState.png)

(2) Promise的结果
```javascript
  const p = new Promise((resolve, reject) => {
    resolve("成功的结果");
  // reject("失败的结果")
})
console.dir(p) 
```

# Promise的方法
(1) then 
+ then的两个参数都是函数, 都是可选的。**它们都是把Promise对象传出的值作为参数**
+ 第一个参数是Promise的状态是fullfilled时执行
+ 第二个参数时Promise的状态是rejected时执行
+ then方法返回一个新的Promise实例，此时状态是pending 
+ Promise的状态不改变就不会执行then里面的方法
+ 在then方法中通过return将返回的Promise实例改成fulfilled状态
  
```javascript
const p = new Promise((resolve, reject) => {
  // resolve("成功的结果");
    reject("失败的结果")
})

const t = p.then((res)=>{
  // 当Promise的状态使fulfilled时执行
  console.log("成功的回调", value)
//   这里的return可以让t实例的状态改成fulfilled
  return 123
},(error)=>{
  // 当Promise的状态时rejected时, 执行
  console.log("失败时调用", error)
})

const q = t.then((res) => {
    console.log('成功', value)
},(error) => {
    console.log('失败', error)
})
console.dir(p) 
```

```javascript
resolve('成功的结果')
reject('失败的结果')
// 而then中的两个函数参数类似于以下
// 因为可以通过参数获取到Promise对象中的结果
const resolve = (res) => {}
const reject = (err) => {}
```

(2) catch
+ 当Promise的状态改为rejected时被执行
+ 当Promise执行过程中出现错误时被执行
```javascript
const p = new Promise((resolve, reject) => {
    reject()
    throw new Error("出错啦")
})

p.catch((error) => {
    console.log('失败', error)
})
```

封装ajax请求, 同时可以理解了axios返回的是一个Promise，
在请求接口后需要处理，可以用then，或者async await获取到返回的数据
```javascript
function getData(url, data = {}){
	return new Promise((resolve, reject) => {
  	$.ajax({
      // 发送请求类型
    	type: "GET",
      url: url,
      data: data,
      success: function (res) {
      	// 修改Promise状态为成功, 修改Promise的结果res
        resolve(res)
      },
      error:function (res) {
      	// 修改Promise的状态为失败,修改Promise的结果res
        reject(res)
      }
    })
  }
}

// 调用函数
getData("data1.json")
  .then((data) => {
  	// console.log(data)
    const { id } = data
    return getData("data2.json", {id})
  })
  .then((data) => {
  	// console.log(data)
    const { usename } = data
    return getData("data3.json", {usename})
  })
  .then((data) => {
  	console.log(data)
  })
```
# Promise的缺点
+ 一旦新建立即执行，无法取消。
+ 如果不设置回调函数，Promise内部抛出错误，不会反应到外部。**Promise会吃掉错误**
+ 当处于pending状态，无法得知目前处于哪个阶段，是刚开始还是快完成


