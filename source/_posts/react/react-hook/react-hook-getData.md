---
title: reat-hook如何获取数据
date: 2022-09-21 00:29
tags: react
---

这是不推荐的一种模式，要记住effect外部的函数使用了哪些props和state很难。
```jsx
function Demo(props) {
  const { query } = props;
  const [list, setList] = useState([]);

  const fetchData = async () => {
    const res = await axios(`/getList?query=${query}`);
    setList(res);
  };

  useEffect(() => {
    fetchData(); // 这样不安全(调用的fetchData函数使用了query)
  }, []);

  return (
    <ul>
      {list.map(({ text }) => {
        return (
          <li key={text}>{ text }</li>
        );
      })}
    </ul>
  );
};
```

通常会想要在effect内部去声明它所需要的函数，这样就能容易的看出那个effect依赖了组件作用域中的哪些值
```jsx
function Demo(props) {
  const { query } = props;
  const [list, setList] = useState([]);

  useEffect(() => {
    const fetchData = async () => {
      const res = await axios(`/getList?query=${query}`);
      setList(res);
    };

    fetchData();
  }, [query]);

  return (
    <ul>
      {list.map(({ text }) => {
        return (
          <li key={text}>{ text }</li>
        );
      })}
    </ul>
  );
};
```

但是如果你在不止一个地方用到了这个函数或者别的原因，你无法把一个函数移动到effect内部，还有一些其他办法：

+ 如果这函数不依赖state、props内部的变量。可以把这个函数移动到你的组件之外。这样就不用其出现在依赖列表中了。
+ 如果其不依赖state、props。但是依赖内部变量，可以将其在effect之外调用它，并让effect依赖于它的返回值。
+ 万不得已的情况下，你可以把函数加入effect的依赖项，但把它的定义包裹进useCallBack。这就确保了它不随渲染而改变，除非它自身的依赖发生了改变。

```jsx
function Demo(props) {
  const { query } = props;
  const [id, setId] = useState();
  const [list, setList] = useState([]);

  const fetchData = async (newId) => {
    const myId = newId || id;
    if (!myId) {
      return;
    }
    const res = await axios(`/getList?id=${myId}&query=${query}`);
    setList(res);
  };

  const fetchId = async () => {
    const res = await axios('/getId');
    return res;
  };

  useEffect(() => {
    fetchId().then(id  => {
      setId(id);
      fetchData(id);
    });
  }, []);

  useEffect(() => {
    fetchData();
  }, [query]);

  return (
    <ul>
      {list.map(({ text }) => {
        return (
          <li key={text}>{ text }</li>
        );
      })}
    </ul>
  );
};
```

解决方法：
```jsx
function Demo(props) {
  const { query } = props;
  const [id, setId] = useState();
  const [list, setList] = useState([]);

  useEffect(() => {
    const fetchId = async () => {
      const res = await axios('/getId');
      setId(res);
    };

    fetchId();
  }, []);

  useEffect(() => {
    const fetchData = async () => {
      const res = await axios(`/getList?id=${id}&query=${query}`);
      setList(res);
    };
    if (id) {
      fetchData();
    }
  }, [id, query]);

  return (
    <ul>
      {list.map(({ text }) => {
        return (
          <li key={text}>{ text }</li>
        );
      })}
    </ul>
  );
}
```