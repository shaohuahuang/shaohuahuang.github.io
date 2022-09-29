## useState是如何实现的

## 前言
在hooks出现之前，React都是通过setState来触发组件的更新。hooks推出之后，我们可以用useState来管理函数组件内部的状态。今天我们来看一下useState是如何实现的。

在进入源码阅读之前，我们先通过下一段代码来看一下useState函数的一些特征。

```javascript
export const Test1 = () => {
  const [count, setCount] = useState(0)
  useEffect(() => {
    setCount(1)
  }, [])
  return (
    <div>
      <p>{count}</p>
    </div>
  )
}
```

首先useState会返回状态的值，可改变状态的方法。对应我们上面的代码就是count和setCount。    

另外一点，我们需要注意的是，函数组件触发更新的时候，函数仍然会被执行一遍。所以第二次执行的时候，useState就不应该是创建hook了，而是得把之前已经创建的对应的hook取出来。


## mount阶段的useState
一个组件的生命周期有mount阶段和update阶段。我们先来看mount阶段，即首次渲染的时候，useState是如何执行的。

```javascript
export function useState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  const dispatcher = resolveDispatcher();
  return dispatcher.useState(initialState);
}
```
这里有一个resolveDispatcher函数，它根据当前阶段是mount阶段还是update阶段，返回对应的dispatcher。如下所示，mount阶段调用的函数是mountUpdate, 而更新阶段调用的是updateState。

```javascript
const HooksDispatcherOnMount: Dispatcher = {
  useState: mountState,
  //other hooks
};

const HooksDispatcherOnUpdate: Dispatcher = {
  useState: updateState,
  //other hooks
};
```

我们现在来看一下mountUpdate的逻辑.

```javascript
function mountState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  const hook = mountWorkInProgressHook();
  if (typeof initialState === 'function') {
    // $FlowFixMe: Flow doesn't like mixed types
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;
  const queue = (hook.queue = {
    pending: null,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: (initialState: any),
  });
  const dispatch: Dispatch<
    BasicStateAction<S>,
  > = (queue.dispatch = (dispatchAction.bind(
    null,
    currentlyRenderingFiber,
    queue,
  ): any));
  return [hook.memoizedState, dispatch];
}

function mountWorkInProgressHook(): Hook {
  const hook: Hook = {
    memoizedState: null,

    baseState: null,
    baseQueue: null,
    queue: null,

    next: null,
  };

  if (workInProgressHook === null) {
    // This is the first hook in the list
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
    // Append to the end of the list
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}

export type Hook = {|
  memoizedState: any,
  baseState: any,
  baseQueue: Update<any, any> | null,
  queue: UpdateQueue<any, any> | null,
  next: Hook | null,
|};
```
如上所示，在mountUpdate里面，首先会调用mountWorkInProgressHook方法，创建一个hook出来。根据Hook的结构，我们知道hook其实最后会形成一个链表。  
每次创建的hook会被放到链表最后。而第一个创建的hook会被赋值给当前Fiber节点的memoizedState。
> 注意： 这里函数组件对应的Fiber，memoizedState是指向hook链表的头节点。而类组件，memoizedState是直接存放组件对应的状态的。

另外我们看一下Hook的数据结构，memoizedState字段其实就是用来存放对应的值。queue是用来存放hook的上面累积的更新，这个和之前的setState是很像的。只不过setState是把更新放在fiber节点的updateQueue里面，而hook是每个节点都维护了自己的queue。这里的queue都是使用的环形链表的数据结构。 

最后我们看一下dispatchAction的实现。每个useState都会返回一个setX的dispatch函数，调用这个函数能够改变组件状态。实现也非常简单。 创建一个udpate对象，把其加入到hook.queue这个链表当中，然后调用scheduleUpdateOnFiber。  

这个dipatch函数会被赋值给queue.dispatch。这样下次再调用useState的时候，可以把对应的dispatch函数取出来。

```javascript
function dispatchAction<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A,
) {
  //...
  const update: Update<S, A> = {
    lane,
    action,
    eagerReducer: null,
    eagerState: null,
    next: (null: any),
  };

  // Append the update to the end of the list.
  const pending = queue.pending;
  if (pending === null) {
    // This is the first update. Create a circular list.
    update.next = update;
  } else {
    update.next = pending.next;
    pending.next = update;
  }
  queue.pending = update;
    //...
    scheduleUpdateOnFiber(fiber, lane, eventTime);
  }
}
``` 

### update阶段的useState

update阶段要做的事情就是把值取出来。调用updateWorkInProgressHook即可找出对应的需要更新的hook。遍历hook.queue，计算出新的状态，然后把值赋给hook.memoizedState。最后返回hook.memoizedState和dispatch

```javascript
function updateState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  return updateReducer(basicStateReducer, (initialState: any));
}

function updateReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  const hook = updateWorkInProgressHook();
  const queue = hook.queue;

  //遍历hook的queue, 计算出newState 

  hook.memoizedState = newState;
  queue.lastRenderedState = newState;

  const dispatch: Dispatch<A> = (queue.dispatch: any);
  return [hook.memoizedState, dispatch];
}
```


### 手写一个useState 

```javascript
let hooks = []
let currentHookIndex = -1
let mounting = true // mount阶段还是update阶段
function useState(initialState){
  let hook;
  currentHookIndex++

  if(mounting){ //mouting阶段创建hook
    hook = [initialState]
    hooks[currentHookIndex] = [initialState, (state) => {
      hooks[currentHookIndex][0] = state
    }]
  }else{ //update阶段取出hook, 更新他的值
    if(currentHookIndex >= hooks.length){ //重复render会使得currentHookIndex越界，这时要回到顶点
      currentHookIndex = 0
    }
    hook = hooks[currentHookIndex]
    let dispatch = hook[1]
    dispatch(typeof initialState === 'function'? initialState() || initialState)
  }
  return hook
}
```

### 总结
- 函数组件执行的时候分为mount阶段和updae阶段，不同的阶段useState的逻辑是不一样的。 mount阶段，执行的是HooksDispatcherOnMount里定义的mountState函数。update阶段，执行的是HooksDispatcherOnUpdate里定义的updateState函数

- 函数组件里的多个useState调用，每次调用都会创建出一个hook节点，这些节点形成一个环状链表。每个节点里面有memoizedState字段和queue字段。memoizedState用于存放hook对应的状态值，queue则会存放这个hook的所有update, 即调用setX函数所触发的更新

- 当前Fiber节点的memoizedState，指向hook链表的头节点。这个与类组件是不太一样的，类组件的memoizedState存的是组件的state值