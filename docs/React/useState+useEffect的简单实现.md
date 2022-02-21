# useState和useEffect的简单实现

## useState
先回忆一下，useState 是怎么调用的？
```
const Comp = props => {
    const [ foo, setFoo ] = useState(1);
    return <button onClick={() => setFoo(2)}>{foo}</button>
};
```

#### useState实现的基本思路：
1. 在组件外部存一个变量存储state的值
2. 通过useState，返回改变量，以及变量的setter函数

一个简单的实现：
```
let currentState;
const useState = initialState => {
  const state = initialState
  const setState = newState => {
    currentState = newState
    render() //触发整个React App的重新刷新
  }
  return [state, setState]
}

```

#### 收纳盒：兼容多次调用

实现思路就是使用数组来存储  
(注：这也是为什么React Hooks不能放在if/for语句里面，因为可能会打破存储顺序，使得获取的值发生错乱)

```
let index = -1
const currentStates = []
const useState = initialState => {
  index++
  const currentIndex = index;
  currentStates[currentIndex] = initialState
  const setState = newState => {
    currentStates[currentIndex] = newState
    render();
  } 
  return [currentStates[currentIndex], setState]
}
```

## useEffect

实现思路：
1. 把dependencies和callback存起来
2. 比较dependencies的变化来决定要不要调用

```
const lastDepsBoxs = [];
const lastClearCallbacks = [];
let index = 0;
const useEffect = (callback, deps) => {
    const lastDeps = lastDepsBoxs[index];
    const changed = !lastDeps || !deps || deps.some((dep, index) => dep !== lastDeps[index]);   
    if (changed) {
        lastDepsBoxs[index] = deps;
        if (lastClearCallbacks[index]) {
            lastClearCallbacks[index]();
        }
        lastClearCallbacks[index] = callback();
    }
    index ++;
};
```