# React重新渲染(rerender)

我最近被下面一段简单的代码搞得有点糊涂，让我对rerender这个词进行了深入理解

```
function App() {
  const [count, setCount] = useState(0)
  const [theme, setTheme] = useState('blue')
  return (
    <div className="App">
        {count}
        <button onClick={() => setCount(count + 1)}>Add</button>
        <ThemeButton theme={theme}></ThemeButton>
    </div>
  );
}

const ThemeButton = ({theme}) => {
  console.log('==1')
  return <button style={{background: theme}}>Theme</button>
}
```

当Add按钮点击的时候，每次ThemeButton都会触发rerender, console.log('==1')会被执行。然后我就想，这样会不会造成DOM的重复渲染了。

仔细想了想，事实说明我是多虑的。ThemeButton的render函数虽然执行了，但是更新DOM是React Diff算法所控制的，ThemeButton里的props并没有发生变化，所以DOM是不会被更新的。

不过这也引出一个问题，虽然昂贵的DOM没有被更新，但是每次render函数都被执行了。如果里面有比较复杂的逻辑，每次开销也挺大的。


## memo
不过好在React提供了memo, memo通过浅层比较props, 来决定需不需要执行render函数。

### 什么时候应该避免用memo？
天下没有免费的午餐，memo虽然可以帮忙跳过一些没有必要的render。但是如果这个组件的props会经常发生变化的，那么memo其实就没有作用了，反而是个累赘，每次都要多进行一次浅层比较。

### callback函数使memo失效的情况
```
function App() {
  const [count, setCount] = useState(0)
  const [theme, setTheme] = useState('blue')
  return (
    <div className="App">
        {count}
        <button onClick={() => setCount(count + 1)}>Add</button>
        <ThemeButton 
          theme={theme}
          onClick=(() => console.log('click'))
        >
        </ThemeButton>
    </div>
  );
}

const ThemeButton = memo({theme}) => {
  console.log('==1')
  return <button style={{background: theme}}>Theme</button>
}
```

ThemeButton里面有个callback函数，App组件每次render的时候，都会创建新的callback，导致ThemeButton的memoization失效。

要解决这个问题，这里就要引入useCallback(fn, deps)这个函数了。这个函数可以对callback进行memoization。  
代码如下

```
function App() {
  const [count, setCount] = useState(0)
  const [theme, setTheme] = useState('blue')
  const clickFunc = useCallback(() => console.log('click'), [])
  return (
    <div className="App">
        {count}
        <button onClick={() => setCount(count + 1)}>Add</button>
        <ThemeButton 
          theme={theme}
          onClick=(clickFunc)
        >
        </ThemeButton>
    </div>
  );
}

const ThemeButton = memo({theme}) => {
  console.log('==1')
  return <button style={{background: theme}}>Theme</button>
}
```


## useMemo
上面提到了memo, 这里就需要再讲一下useMemo。
```
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

useMemo其实就是把一个计算结果存储起来，避免重复计算。这个可以和memo结合在一起使用，有效的避免组件的重复渲染。
























