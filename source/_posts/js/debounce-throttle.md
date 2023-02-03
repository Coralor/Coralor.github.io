---
title: 防抖和节流
date: 2022-12-28 22:33:26
tags: js
---

# 防抖

单位时间内，频繁触发事件，**只执行最后一次**。

例如： 当触发事件时，设定在1000ms后执行，但是在还剩下500ms时你又触发了该事件，那就会重新开始，知道又1000ms后才执行。

核心：从新开始（联想多米诺骨牌，推到只能重新排过）

应用场景： 搜索框的搜索输入，文本编辑器的实时保存

代码实现：利用定时器，每次触发时先清空定时器，重新开始
```javascript
let timer = null
document.querySelector('.button').onkeyup = function() {
    if (timer !== null) {
        clearTimeout(timer)  // 清空定时器，重新来过
    }
    timer = setTimeout(() => {
        console.log('防抖')
    }, 1000)
}
```

# 节流

单位时间内，频繁触发事件，**只执行一次**。

例如：设定1000ms执行，但是你在1000ms中触发了多次，也只是在1000ms后执行一次

核心：不要打断

应用场景：高频事件例如快速点击、鼠标滑动、resize事件，滚动事件，下拉加载，视频播放记录时间

代码实现：也是利用定时器，等待定时器执行完，才开启定时器。

```javascript
let timer = null
document.querySelector('.button').onkeyup = function() {
    if (timer !== null) {
        return   // 阻止再执行
    }
    timer = setTimeout(() => {
        console.log('节流')
        timer = null // 执行完后清空定时器，下次触发事件再开启
    }, 1000)
}
```
