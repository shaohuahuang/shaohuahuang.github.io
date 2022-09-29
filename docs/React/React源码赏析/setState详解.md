## setState详解

### 前言
setState是React里面的一个非常重要的API, 在hooks出现之前，这是唯一的可以更改组件状态的方法。这可以算的上是React的一个基石级别的API。今天我们通过源码仔细看一下它是如何工作的。

通过这篇文章我们要回答以下问题
1. setState如何收集所有的更新，并最后计算出最终状态？
2. setState在默认情况下是异步执行，那么什么时候会出现同步执行的情况？

### 状态收集与计算
说到状态的收集，我们第一时间会想到用一个数组来收集所有的更新。React其实是在FiberNode上面定义了一个updateQueue字段，这个字段对应的是一个循环链表。每次遇到更新的时候，会把新的更新插入到循环链表中。我们来看看源码中的逻辑是什么样子的

```javascript
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  this.tag = tag; // 3 为 HostRoot
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;

  // Fiber
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;

  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  // Effects
  this.flags = NoFlags;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  this.alternate = null;
 }
```

#### 收集
Component类的setState里面会调用enqueueSetState, 然后再调用enqueueUpdate, 把更新插入到updateQueue中。如下所示：
```javascript
Component.prototype.setState = function(partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};

enqueueSetState(inst, payload, callback) {
  const fiber = getInstance(inst);
  const eventTime = requestEventTime();
  const lane = requestUpdateLane(fiber);

  const update = createUpdate(eventTime, lane);
  update.payload = payload;
  if (callback !== undefined && callback !== null) {
    update.callback = callback;
  }

  enqueueUpdate(fiber, update);
  scheduleUpdateOnFiber(fiber, lane, eventTime);
},


export function enqueueUpdate<State>(fiber: Fiber, update: Update<State>) {
  const updateQueue = fiber.updateQueue;
  if (updateQueue === null) {
    // Only occurs if the fiber has been unmounted.
    return;
  }

  const sharedQueue: SharedQueue<State> = (updateQueue: any).shared;
  const pending = sharedQueue.pending;
  if (pending === null) {
    // This is the first update. Create a circular list.
    update.next = update;
  } else {
    update.next = pending.next;
    pending.next = update;
  }
  sharedQueue.pending = update;
}
```

#### 计算
状态的计算实在processUpdateQueue这个函数中, 如下所示

```javascript
export function processUpdateQueue<State>(
  workInProgress: Fiber,
  props: any,
  instance: any,
  renderLanes: Lanes,
): void {
  // This is always non-null on a ClassComponent or HostRoot
  const queue: UpdateQueue<State> = (workInProgress.updateQueue: any);
  //一些初始化工作

  // These values may change as we process the queue.
  if (firstBaseUpdate !== null) {
    // Iterate through the list of updates to compute the result.
    let newState = queue.baseState;
    //...
    do {
      const updateLane = update.lane;
      const updateEventTime = update.eventTime;
      if (!isSubsetOfLanes(renderLanes, updateLane)) {
        // 如果更新的优先级 < 当前渲染的优先级， 则跳过此更新
      } else {
        // This update does have sufficient priority.
        //...

        // Process this update.
        newState = getStateFromUpdate(
          workInProgress,
          queue,
          update,
          newState,
          props,
          instance,
        );
        //...
      }
      update = update.next;
      //...
    } while (true);

    if (newLastBaseUpdate === null) {
      newBaseState = newState;
    }
    //...
    workInProgress.memoizedState = newState;
  }
}

```

在这个函数中，主要做的事情就是通过while循环遍历updateQueue, 如果更新的优先级小于当前渲染的优先级，就不处理。如果大于的话，则会调用getStateFromUpdate, 计算出新的newState. while循环完毕之后，会把最终状态赋值给fiber节点的memoizedState。 

```javascript
function getStateFromUpdate<State>(
  workInProgress: Fiber,
  queue: UpdateQueue<State>,
  update: Update<State>,
  prevState: State,
  nextProps: any,
  instance: any,
): any {
  switch (update.tag) {
    case ReplaceState: {
      //...
    }
    case CaptureUpdate: {
      //...
    }
    // Intentional fallthrough
    case UpdateState: {
      const payload = update.payload;
      let partialState;
      if (typeof payload === 'function') {
        partialState = payload.call(instance, prevState, nextProps);
      } else {
        // Partial state object
        partialState = payload;
      }
      if (partialState === null || partialState === undefined) {
        // Null and undefined are treated as no-ops.
        return prevState;
      }
      // Merge the partial state and the previous state.
      return Object.assign({}, prevState, partialState);
    }
    case ForceUpdate: { 
      //...
    }
  }
  return prevState;
}
```
getStateFromUpdate的逻辑非常简单，计算出partialState, 然后把partialState合并到prevState。
至此setState的逻辑就大致讲完了


### 异步执行和同步执行

接下来我们聊聊什么是同步执行，什么是异步执行了？我们通过下面的例子，来了解一下

```javascript
// 异步执行
export class Test1 extends React.Component{
  state = {count: 0}
  componentWillMount(){
    this.setState({count: 1})
    console.log(this.state.count)
    this.setState({count: 2})
  }

  render(){
    return (
      <div>{this.state.count}</div>
    )
  }
}

// 同步执行
export class Test1 extends React.Component{
  state = {count: 0}
  componentWillMount(){
    setTimeout(() => {
      his.setState({count: 1})
      console.log(this.state.count)
      this.setState({count: 2})
    })
  }

  render(){
    return (
      <div>{this.state.count}</div>
    )
  }
}
```
在上面这段代码中，第一个例子中，setState是异步执行的，所以所以打印出来的是0。第二例子是同步执行的，所以打印出来的是1。 

我们接下来看一下React里面是如何判断setState是需要同步执行还是异步执行的。在最开始的那段代码中，我们可以看到，每次enqueueUpdate之后，就立马调用了scheduleUpdateOnFiber。

```javascript
export function scheduleUpdateOnFiber(
  fiber: Fiber,
  lane: Lane,
  eventTime: number,
) {

  //...
  if (lane === SyncLane) {
    if (
      // Check if we're inside unbatchedUpdates
      (executionContext & LegacyUnbatchedContext) !== NoContext &&
      // Check if we're not already rendering
      (executionContext & (RenderContext | CommitContext)) === NoContext
    ) {
      //...
      performSyncWorkOnRoot(root);
    } else {
      //...
    }
  }
  else {
    // 处理并发模式下的逻辑
    // Schedule other updates after in case the callback is sync.
    ensureRootIsScheduled(root, eventTime);
    schedulePendingInteractions(root, lane);
  }
}
```
在这个函数中，当lane是SyncLane的时候，并且当前的context是LegacyUnbatchedContext的时候，会调用performSyncWorkOnRoot，这个方法会把当前的state立即更新到fiber的状态上。

如果我们的渲染是调用的ReactDOM.render，那么这时的lane就是SyncLane。如果我们调用的是createRoot，那么此时使用了React 18推出的并发模式。

在非并发模式下，分为两种情况：
- 在组件生命周期或React合成事件中，setState是异步；
- 在setTimeout或者原生dom事件中，这个context会被React标记为LegacyUnbatchedContext, setState是同步； 

而如果是并发模式，setState则全部为异步执行。


