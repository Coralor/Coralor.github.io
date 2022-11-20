---
title: useMemo和useCallback
date: 2022-09-15 21:57
tags: react
---

# React的"re-render"是什么？

React利用了"re-render"，使得页面可以根据状态的改变而发生同步的渲染。
看到一段话很形象地描述了该过程：Each re-render is a snapshot of what the application's UI should look like at a given moment in time, based on the current application state. We can think of it like a stack of photographs, each one capturing how things looked given a specific value for every state variable.
**react like playing a "find the differences" game.**We don't directly tell React which DOM nodes need to change. Instead, we tell React what the UI should be based on the current state. By re-rendering, React creates a new snapshot, and it can figure out what needs to change by comparing snapshots.

useMemo和useCallback，使用useMemo和useCallback一定能达到优化的效果吗？为什么能？又为什么不能？

useMemo和useCallback是react的一种记忆函数，利用闭包缓存上次的结果，该函数的成本在于额外的内存和比较逻辑，
最重要的是这不是一种绝对的优化，而是一种成本比较上的优化。


## 为什么使用useMemo和useCallback？
optimize re-renders(优化重复渲染)
1.Reducing the amount of work that needs to be done in a given render. (减少在每次渲染中需要做的工作。)
2.Reducing the number of times that a component needs to re-render. (减少一个组件重复渲染的次数)

## useMemo  
This is commonly known as memoization, and it's why this hook is called “useMemo”.
**useMemo缓存的是arrays / objects.**有助于避免在每次渲染时都进行高开销的计算

useMemo有两个参数：第一个参数为计算过程(回调函数，必须返回一个结果)，第二个参数是依赖项(数组)，当依赖项中某一个发生变化，结果将会重新计算。

### 什么情况下可以使用useMemo?
1.Heavy computations(有繁重的计算)
useMemo is essentially like a lil’ cache, and the dependencies are the cache invalidation strategy.

for example:
我们有两个状态：selectedNum and time。我们根据输入的数字num计算出1-num之间的素数个数。每一秒，这个时间time都会不断地更新它的当前时间。

```jsx
import React from 'react';
import format from 'date-fns/format';

function App() {
  const [selectedNum, setSelectedNum] = React.useState(100);

  const time = useTime();
  
  const allPrimes = [];
  for (let counter = 2; counter < selectedNum; counter++) {
    if (isPrime(counter)) {
      allPrimes.push(counter);
    }
  }
  
  return (
    <>
      <p className="clock">
        {format(time, 'hh:mm:ss a')}
      </p>
      <form>
        <label htmlFor="num">Your number:</label>
        <input
          type="number"
          value={selectedNum}
          onChange={(event) => {
            let num = Math.min(100_000, Number(event.target.value));
            setSelectedNum(num);
          }}
        />
      </form>
      <p>
        There are {allPrimes.length} prime(s) between 1 and {selectedNum}:
        {' '}
        <span className="prime-list">
          {allPrimes.join(', ')}
        </span>
      </p>
    </>
  );
}

// 创建时钟的函数
function useTime() {
  const [time, setTime] = React.useState(new Date());
  
  React.useEffect(() => {
    const intervalId = window.setInterval(() => {
      setTime(new Date());
    }, 1000);
  
    return () => {
      window.clearInterval(intervalId);
    }
  }, []);
  
  return time;
}

// 判断是否为素数的函数
function isPrime(n){
  const max = Math.ceil(Math.sqrt(n));
  
  if (n === 2) {
    return true;
  }
  
  for (let counter = 2; counter <= max; counter++) {
    if (n % counter === 0) {
      return false;
    }
  }

  return true;
}

export default App;
```
!['进行繁重的计算'](/images/heavy-computations.png)

现在的问题是： 不管哪个状态发生改变，我们都要重新计算素数的个数。并且，因为时钟是每一秒都在发生变化的，所以我们要持续不断得重新生成一连串得素数，即使我们输入的数字并没有发生改变。那为什么我们不“skip” these calculations呢？我们可以re-use之前已经计算好的素数，而不是在每一秒都抓狂地重复计算。

因此，我们可以利用useMemo进行optimize

```jsx
const allPrimes = React.useMemo(() => {
  const result = [];
  for (let counter = 2; counter < selectedNum; counter++) {
    if (isPrime(counter)) {
      result.push(counter);
    }
  }
  return result;
}, [selectedNum]);
```

使用了useMemo以后，我们只有在selectedNum发生变化时才会重复计算这范围内的素数。而这组件由于其他原因而发生的重新渲染，都不会再重复计算，而是使用先前缓存的值。

除了useMemo,还能怎么减少不必要的重复计算呢？
use pure components, memo().This means that it should only re-render whenever its props change.

Our PurePrimeCalculator will only re-render when it receives new data, or when its internal state changes.

2.Preserved references(需要缓存地址)
```jsx
// App.js
import React from 'react';

import Boxes from './Boxes';

function App() {
  const [name, setName] = React.useState('');
  const [boxWidth, setBoxWidth] = React.useState(1);
  
  const id = React.useId();
  
  // Try changing some of these values!
  const boxes = [
    { flex: boxWidth, background: 'hsl(345deg 100% 50%)' },
    { flex: 3, background: 'hsl(260deg 100% 40%)' },
    { flex: 1, background: 'hsl(50deg 100% 60%)' },
  ];
  
  return (
    <>
      <Boxes boxes={boxes} />
      
      <section>
        <label htmlFor={`${id}-name`}>
          Name:
        </label>
        <input
          id={`${id}-name`}
          type="text"
          value={name}
          onChange={(event) => {
            setName(event.target.value);
          }}
        />
        <label htmlFor={`${id}-box-width`}>
          First box width:
        </label>
        <input
          id={`${id}-box-width`}
          type="range"
          min={1}
          max={5}
          step={0.01}
          value={boxWidth}
          onChange={(event) => {
            setBoxWidth(Number(event.target.value));
          }}
        />
      </section>
    </>
  );
}

export default App;
```

```jsx
// Boxes.js
import React from 'react';

function Boxes({ boxes }) {
  return (
    <div className="boxes-wrapper">
      {boxes.map((boxStyles, index) => (
        <div
          key={index}
          className="box"
          style={boxStyles}
        />
      ))}
    </div>
  );
}

export default React.memo(Boxes);

```

在本例子中，Boxes组件是pure component,它只有在props发生变化才会re-render。
现在的问题在于，Boxes组件只有一个props，即boxes， 但是boxes在每次重新渲染时都是相同数据的数组。然而，在每一次re-render时，我们都要生成新的数组，而这些数组的存储的数据都是一样，但是它们的引用地址（reference）却是不一样的。因此会进行很多不必要的渲染。

为什么每次的boxes看似一模一样，却每次都要重新创建数组呢？
**the boxes prop has received a freshly-created, never-before-seen array.**

```js
function getNumbers() {
  return [1, 2, 3];
}
const firstResult = getNumbers();
const secondResult = getNumbers();
console.log(firstResult === secondResult);  //false 

```
 很形象地说法是，Two identical twins are not the same person.
 !['进行繁重的计算'](/images/different-people.png)

对于strings, numbers, boolean这些数据类型，是根据值本身去比较是否相同的，
但是对于array, object的数据类型，是只能根据它们本身的地址引用去比较的。

解决方法：
```jsx
const boxes = React.useMemo(() => {
  return [
    { flex: boxWidth, background: 'hsl(345deg 100% 50%)' },
    { flex: 3, background: 'hsl(260deg 100% 40%)' },
    { flex: 1, background: 'hsl(50deg 100% 60%)' },
  ];
}, [boxWidth]);
```
这次使用useMemo并非像上一个例子那样是为了减少素数的计算次数。
Our only goal is to preserve a reference to a particular array. Ignoring renders that don't affect the UI.
 !['使用useMemo前'](/images/useMemo-before.png)

 !['使用useMemo后'](/images/useMemo-after.png)

## useCallback
useCallback的使用几乎与useMemo一样，**不过useCallback缓存的是一个function**，当依赖项中的一项发现变化，函数体会重新创建。

```js
// Similar to arrays and objects, functions are compared by reference, not by value.
const functionOne = function() {
  return 5;
};
const functionTwo = function() {
  return 5;
};
console.log(functionOne === functionTwo); // false
```
也就是说，如果我们在组件中定义了一个函数，那么在组件的每次重新渲染都会导致re-generated该函数
（an identical-but-unique function）

```jsx
// App.js
import React from 'react';

import MegaBoost from './MegaBoost';

function App() {
  const [count, setCount] = React.useState(0);

  function handleMegaBoost() {
    setCount((currentValue) => currentValue + 1234);
  }

  return (
    <>
      Count: {count}
      <button
        onClick={() => {
          setCount(count + 1)
        }}
      >
        Click me!
      </button>
      <MegaBoost handleClick={handleMegaBoost} />
    </>
  );
}

export default App;

```

```jsx
// megaBoost.js
import React from 'react';

function MegaBoost({ handleClick }) {
  console.log('Render MegaBoost');
  
  return (
    <button
      className="mega-boost-button"
      onClick={handleClick}
    >
      MEGA BOOST!
    </button>
  );
}

export default React.memo(MegaBoost);
```
MegaBoost组件是个pure component, 它并不依赖于count的变化，但是每一次count的改变都是导致它re-render。Why?
因为每次count的改变都是使得组件重新渲染，而App component重新渲染会重新创建handleMegaBoost函数，这就使得MegaBoost component的props发生改变，因此MegaBoost 也会重新渲染。

那怎么解决呢？
1. 使用useMemo解决
```js
const handleMegaBoost = React.useMemo(() => {
  return function() {
    setCount((currentValue) => currentValue + 1234);
  }
}, []);
```
2. 使用useCallback解决
```js
const handleMegaBoost = React.useCallback(() => {
  setCount((currentValue) => currentValue + 1234);
}, []);
```

实际上，useCallback和useMemo的功能很像，只是useCallback缓存的是函数，而useMemo缓存的是值
```js
React.useCallback(function helloWorld(){}, []);
// equivalent to this:
React.useMemo(() => function helloWorld(){}, []);
```

## 什么时候使用useMemo和useCallback更合适?

一个函数执行完毕之后，就会从函数调用栈中被弹出，里面的内存也会被回收。因此，即使在函数内部创建了多个函数，执行完毕之后，这些创建的函数也都会被释放掉。函数式组件的性能是非常快的。相比class，函数更轻量，也避免了使用高阶组件、renderProps等会造成额外层级的技术。使用合理的情况下，性能几乎不会有什么问题。

而当我们使用useMemo/useCallback时，由于新增了对于闭包的使用，新增了对于依赖项的比较逻辑，因此，盲目使用它们，甚至可能会让你的组件变得更慢。大多数情况下，这样的交换，并不划算。你的组件可能并不需要使用useMemo/useCallback来优化。

**总的来说，就是需要权衡组件re-render 的次数和React.memo、useMemo与useCallback这些缓存机制的代价，做好平衡，不能盲目的多用这类缓存优化方案**

通常情况下，当函数体或者结果的计算过程非常复杂时，我们才会考虑优先使用useCallback/useMemo。

例如在一个一定会多次re-render的组件里，input的回调没有任何依赖项，我们就可以使用useCallback来降低多次执行带来的重复创建同样方法的负担。
又例如，在日历组件中，需要根据今天的日期，计算出当月的所有天数以及相关的信息。

**不过，当依赖项会频繁变动时，我们也要考虑使用useMemo/useCallback是否划算。**
