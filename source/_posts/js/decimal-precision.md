---
title: js小数精度处理
date: 2022-12-28 09:35:24
tags: js
---

# toFixed()
```javascript
    const num =2.446242342;
    num = num.toFixed(2);  // 输出结果为 2.45
```

像 round()、floor()、ceil() 等都不能真正的四舍五入，有精度问题。但是round可以通过以下方式确保精度。

# round()
```javascript
    const num =2.446242342;
    num = Math.round((num + Number.EPSILON) * 100) / 100;  // 输出结果为 2.45
```

