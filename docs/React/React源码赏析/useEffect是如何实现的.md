## useEffect是如何实现的

### 前言
useEffect是组件函数处理副作用的地方。所谓副作用就是对外部环境有依赖的逻辑处理，譬如网络请求，数据读取等。 

我们先通过下面一段简单的代码，来看看useEffect的特性。
```javascript
useEffect(() => {
  elem.addEventLister('click', () => { conosle.log('clicked')})
  return elem.removeEventListener
}, deps)
```

useEffect接受两个参数，第一个参数是effect副作用函数，第二个是依赖数组。
- effect副作用函数会在commmit阶段之后执行。effect函数的返回值是一个销毁函数
- deps数组是用来与上一次的数组比较，浅层比较之后，来决定effect需不需要被执行。当前deps为空数组的时候，那么effect函数只会在首次mount的时候执行，useEffect的作用就好像componentDidMount

了解完特性之后，我们今天源码阅读的目标主要弄清楚以下几点： 
- useEffect创建的hook节点的数据结构是什么样子，存储在什么地方？
- 如何根据deps数组来判断effect函数需不需要被执行？
- 销毁函数是什么时候被执行的


### mount阶段的useEffect 

```javascript
// react/packages/react/src/ReactHooks.js
export function useEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null,
): void {
  const dispatcher = resolveDispatcher();
  return dispatcher.useEffect(create, deps);
}

function mountEffectImpl(fiberFlags, hookFlags, create, deps): void {
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  currentlyRenderingFiber.flags |= fiberFlags;
  hook.memoizedState = pushEffect(
    HookHasEffect | hookFlags,
    create,
    undefined,
    nextDeps,
  );
}

function pushEffect(tag, create, destroy, deps) {
  const effect: Effect = {
    tag,
    create,
    destroy,
    deps,
    // Circular
    next: (null: any),
  };
  //...
  let componentUpdateQueue: null | FunctionComponentUpdateQueue = (currentlyRenderingFiber.updateQueue: any);
  if (componentUpdateQueue === null) {
    componentUpdateQueue = createFunctionComponentUpdateQueue();
    currentlyRenderingFiber.updateQueue = (componentUpdateQueue: any);
    componentUpdateQueue.lastEffect = effect.next = effect;
  }else{
    const lastEffect = componentUpdateQueue.lastEffect;
    const firstEffect = lastEffect.next;
    lastEffect.next = effect;
    effect.next = firstEffect;
    componentUpdateQueue.lastEffect = effect;
  }
  //...
  return effect;
}
```
mount阶段会调用mountEffect函数，mountEffect里面会调用mountEffectImpl。  
这个函数调用mountWorkInProgressHook，这里其实与useState一样，创建hook节点，把节点放入到链表。我们可以看到，React其实是把所有hook节点放在同一个链表上的。 

然后mountEffectImpl会调用pushEffect函数。这个函数其实就是创建Effect节点，然后把节点放入到effects链表中。这个effects链表是存储在当前Fiber节点的updateQueue里面。  

最后hook.memoizedState会指向创建出来的effect的节点
> 这里我们可以发现，React函数组件的Fiber节点的updateQueue是用来存放effects列表的，而类组件的updateQueue是收集setState所触发的所有更新的。  
>
> 另一点需要注意的是，useEffect的信息是被分别存储到hook节点和effect节点。之所以要这么做，应该是因为React想要把所有不同hooks的节点都放在同一条链表上，但是执行effect函数的时候，又不想遍历整个hooks链表，所以单独创建一个effects链表。 


### update阶段的useEffect 

```javascript
function updateEffectImpl(fiberFlags, hookFlags, create, deps): void {
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  let destroy = undefined;

  if (currentHook !== null) {
    const prevEffect = currentHook.memoizedState;
    destroy = prevEffect.destroy;
    if (nextDeps !== null) {
      const prevDeps = prevEffect.deps;
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        pushEffect(hookFlags, create, destroy, nextDeps);
        return;
      }
    }
  }

  currentlyRenderingFiber.flags |= fiberFlags;

  hook.memoizedState = pushEffect(
    HookHasEffect | hookFlags,
    create,
    destroy,
    nextDeps,
  );
}

function areHookInputsEqual(
  nextDeps: Array<mixed>,
  prevDeps: Array<mixed> | null,
) {
  if (prevDeps === null) {
    return false;
  }

  for (let i = 0; i < prevDeps.length && i < nextDeps.length; i++) {
    if (is(nextDeps[i], prevDeps[i])) {
      continue;
    }
    return false;
  }
  return true;
}
```

updateEffect会调用updateEffectImpl, 通过当前hook节点取出effect节点, 然后比较prevDeps和nextDeps，来决定是否需要把flag标记为HookHasEffect。标记为HookHasEffect的，在执行阶段的时候会得到执行。

deps的比较逻辑在areHookInputsEqual函数中，是对两个数组的浅比较。如果deps为null的话，直接返回false

> 这里需要注意的是pushEffect每次都需要执行的。这是因为每次更新的时候，fiber的updateQueue已经被置为的null, 所以effects链表需要重新生成。为了能够在销毁阶段调用到销毁函数，deps即使一样，也需要把effect节点放入effects链表。只是如果deps是一样的，那么hook.memoizedState不需要重新赋值。


### 执行阶段
在commitLifeCycles函数中，会调用schedulePassiveEffects遍历effects列表，如果有HookHasEffect标签，则把他们都加入到两个数组中pendingPassiveHookEffectsUnmount, pendingPassiveHookEffectsMount。如下所示：

```javascript
function schedulePassiveEffects(finishedWork: Fiber) {
  const updateQueue: FunctionComponentUpdateQueue | null = (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
      const {next, tag} = effect;
      if (
        (tag & HookPassive) !== NoHookEffect &&
        (tag & HookHasEffect) !== NoHookEffect
      ) {
        enqueuePendingPassiveHookEffectUnmount(finishedWork, effect);
        enqueuePendingPassiveHookEffectMount(finishedWork, effect);
      }
      effect = next;
    } while (effect !== firstEffect);
  }
}
```

在DOM渲染完毕之后，会调用flushPassiveEffects来执行所有的effects
```javascript
// First pass: Destroy stale passive effects.
  const unmountEffects = pendingPassiveHookEffectsUnmount;
  pendingPassiveHookEffectsUnmount = [];
  for (let i = 0; i < unmountEffects.length; i += 2) {
    const effect = ((unmountEffects[i]: any): HookEffect);
    const fiber = ((unmountEffects[i + 1]: any): Fiber);
    const destroy = effect.destroy;
    effect.destroy = undefined;

    if (typeof destroy === 'function') {
        destroy();
    }
  }
  // Second pass: Create new passive effects.
  const mountEffects = pendingPassiveHookEffectsMount;
  pendingPassiveHookEffectsMount = [];
  for (let i = 0; i < mountEffects.length; i += 2) {
    const effect = ((mountEffects[i]: any): HookEffect);
    const fiber = ((mountEffects[i + 1]: any): Fiber);
    const create = effect.create;
    effect.destroy = create();
  }
```

遍历unmountEffects数组的时候会调用destroy函数，把一些副作用譬如监听事件去掉。
遍历mountEffects数组的时候会调用create函数，并把destroy赋值给effect节点。 


### 手写useEffect
这里我们假设 effect是存储在hook上面的

```javascript
let hooks = [] // Array<{create, destroy, deps, flag}> | flag用于判断执行阶段hook需不需要被执行
let currentHookIndex = -1 
let mounting = true 

const useEffect = (create, deps){
  currentHookIndex++
  if(mouting){ //mount阶段
    hooks.push({
      create,
      destroy: undefined,
      deps，
      flag: true
    })
  }else{ //update阶段 
    currentHookIndex = currentHookIndex % hooks.length // update阶段的时候需要从头开始
    let hook = hooks[currentHookIndex]
    let prevDeps = hook.deps

    //判断deps是否相同
    let equal = true;
    if(prevDeps == null)
      equal = false
    else{
      for(let i=0; i< deps.length; i++){ //遍历deps数组
        if(prevDeps[i] !== deps[i])
          equal = false
      }
    }

    //不相同的话，需要先销毁，然后创建新节点
    if(!equal){
      if(hook.destroy) hook.destroy() // 把之前的给销毁掉
      hooks[currentHookIndex] = {
        create,
        destroy: undefined
        deps,
        flag: true
      }
    }else{
      hook.flag = false
    }
  }
}

//执行阶段
const flushEffects = () => {
  hooks.forEach(hook => {
    if(hook.flag) //只执行标记为执行的hook
      hook.destroy = hook.create()
  })
}

```

### useEffect里的setTimeout回调为什么无法取到最新的state？ 
我们通过下面这一段代码来看理解一下这个问题。 useEffect里面有个setTimeout, 1s后会把count打印出来。再打印之前，我点击了div，让count增加为1。然而setTimeout打印出来的却是0。

```javascript
const App = () => {
  const [count, setCount] = useState(0)
  useEffect(() => {
    setTimeout(() => {
      console.log(count)
    }, 1000)
  }，[])

  return <div onClick={() => setCount(count + 1)}>{count}</div>
}
```

这个问题如果我们不了解useEffect源码的时候，是非常难解释的。现在我们了解源码之后，就很好解释。
我们知道useEffect会创建effect节点，并把create函数放入到节点中。App组件函数第一次调用的时候，count为0， 并把create函数存起来。这个时候因为create没有被销毁，所以就形成了一个闭包，所以这里的count就是指向这个闭包里的值。即使以后调用setCount发起更新，但是闭包里的count的值引用并不会发生变化。所以打印出来的仍然为0。 


### 总结
- mount阶段useEffect会分别创建hook节点和effect节点。hook节点会加入到hooks链表中，effect节点会加入到effects链表中。hooks链表的头节点被赋值给fiber.memoizedState, 而effects链表的头节点被赋值给fiber.updateQueue

- update阶段，会重新创建effect节点。如果deps不相同，则把effect标记为HookHasEffect，hook.memoizedState会指向这个新的effect节点。

- 执行阶段遍历effects列表时会跳过不带有HookHasEffect标签的元素