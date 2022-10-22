## Node里面的EventLoop

```
    本阶段执行已经被 setTimeout() 和 setInterval() 的调度回调函数。
   ┌───────────────────────────┐
┌─>│           timers          │ 
│  └─────────────┬─────────────┘
|   执行延迟到下一个循环迭代的 I/O 回调。
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
|   仅系统内部使用。
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      
|   检索新的I/O事件;执行与 I/O 相关的回调  ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│   setImmediate() 回调函数在这里执行。  └───────────────┘
│  ┌─────────────┴─────────────┐      
│  │           check           │
│  └─────────────┬─────────────┘
|  一些关闭的回调函数
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  
   └───────────────────────────┘
```

通过 Node.js 的官方文档可以得知，其事件循环的顺序分为以下六个阶段，每个阶段都会处理专门的任务： 
- timers： 计时器阶段，用于处理setTimeout以及setInterval的回调函数
- pending callbacks： 用于执行某些系统操作的回调，例如TCP错误
- idle, prepare： Node内部使用，不用做过多的了解
- poll： 轮询阶段，执行队列中的 I/O 队列，并检查定时器是否到时
- check： 执行setImmediate的回调
- close callbacks： 处理关闭的回调，例如 socket.destroy()

以上六个阶段，我们需要重点关注的只有四个，分别是 **timers 、poll 、check 、close callbacks**

这四个阶段都有**各自的宏队列**，只有当本阶段的宏队列中的任务处理完以后，才会进入下一个阶段  
在执行的过程中会不断检测微队列中是否存在待执行任务，若存在，则执行微队列中的任务，等到微队列为空了，再执行宏队列中的任务（这一点与浏览器非常类似)

|名称|常用|
|----|----|
|宏任务|setTimeout 、setInterval 、setImmediate|
|微任务|Promise 、process.nextTick|  

可以看到，在Node.js对比浏览器多了两个任务，分别是宏任务 `setImmediate` 和 `微任务 process.nextTick`

setImmediate 会在 **check** 阶段被处理
process.nextTick 是Node.js中一个特殊的微任务，因此会为它单独提供一个队列，称为 next tick queue，**并且其优先级大于其它的微任务**，即若同时存在 process.nextTick 和 promise，则会先执行前者

总结一下，Node.js在事件循环中涉及到了4个宏队列和2个微队列，如图所示
![](./images/queues.awebp)


最后看一下代码的输出
```javascript
setTimeout(() => {
    console.log(1);
}, 0)

setImmediate(() => {
    console.log(2);
})

new Promise(resolve => {
    console.log(3);
    resolve()
    console.log(4);
})
.then(() => {
    console.log(5);
})

console.log(6);

process.nextTick(() => {
    console.log(7);
})

console.log(8);

/* 打印结果：
			3
			4
			6
			8
			7
			5
			1
			2
*/
```