# React Hooks闭包陷阱

在使用React Hooks的时候, 有的时候我很惊讶为什么hooks函数里面取不到组件state最新的值。今天我们就来研究一下这个问题。

我们先来看一下下面这段代码。在2s之内多次点击Increase按钮，useEffect代码里的setTimeout函数输出的都是0。
```
function WatchCount() {
  const [count, setCount] = useState(0);

  useEffect(function () {
    setTimeout(function log() {
      console.log(`Count is: ${count}`);
    }, 2000);
  }, []);

  return (
    <div>
      {count}
      <button onClick={() => setCount(count + 1)}>Increase</button>
    </div>
  );
}

```

我们从函数的[词法环境](JS基础知识/词法环境.md)来解释为什么会这样。
点击Increase按钮，每次都会触发React组件的刷新，也就是说WatchCount每次都会被调用。但是useEffect的依赖是空数组，所以其实只会在第一次时候的执行。

我们看第一次执行的时候词法环境链条长什么样子。
```
// LexicalEnvironment of log()
LexicalEnvironment = {
  outer = LexicalEnvironment of setTimeout()
}

// LexicalEnvironment of setTimeout()
LexicalEnvironment = {
  outer = LexicalEnvironment of useEffect()
}

// LexicalEnvironment of useEffect()
LexicalEnvironment = {
  outer = LexicalEnvironment of WatchCount()
}

// LexicalEnvironment of WatchCount()
LexicalEnvironment = {
  count = 0
  outer = global LexicalEnvironment
}
```

顺着这条链条查上去，会发现count = 0。
这里要特别注意一下，虽然WatchCount被多次调用，但是因为useEffect只在第一次的时候被调用，所以setTimeout里的log函数的外部词法环境，永远引用那个时候创建的词法环境。

## 更通俗的理解
每次调用WatchCount都会形成一个闭包，因为useEffect只会在第一次调用，所以他的内部只能获取到第一个闭包里的count的值。第一个闭包的count是0，所以最后输出的是0。


## 如何打印出最新的count值呢？
这个时候需要使用useRef。
#### useRef官方定义
> useRef returns a mutable ref object whose .current property is initialized to the passed argument (initialValue). The returned object will persist for the full lifetime of the component.  

useRef返回的是挂载在当前ReactNode的memoizedState对象。ref.current可以获取到这个对象里的值。  
可以这样来理解，useState在WatchCount的外部创建了一个count变量，useRef在外部创建了一个对象，每次获取这个对象的current属性，都会返回count最新的值。
修改后的代码如下所示：

```
function WatchCount() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);
  countRef.current = count;

  useEffect(function () {
    setTimeout(function log() {
      // console.log(`Count is: ${count}`); 
      console.log(`Count is: ${countRef.current}`);
    }, 2000);
  }, []);

  return (
    <div>
      {count}
      <button onClick={() => setCount(count + 1)}>Increase</button>
    </div>
  );
}

```


Reference: 
1. [简单聊一聊 hooks 与闭包](https://zhuanlan.zhihu.com/p/95947450?utm_source=wechat_timeline&utm_medium=social&utm_oi=69668794531840&utm_campaign=shareopn)
2. https://dmitripavlutin.com/react-hooks-stale-closures/
3. [useEffect的全部指南 - Dan](https://overreacted.io/a-complete-guide-to-useeffect/)