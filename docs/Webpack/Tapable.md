## Tapable介绍

Tapable 是 Webpack 插件架构的核心支架，但它的源码量其实很少，本质上就是围绕着 订阅/发布 模式叠加各种特化逻辑，适配 webpack 体系下复杂的事件源-处理器之间交互需求，比有些场景需要支持将前一个处理器的结果传入下一个回调处理器；有些场景需要支持异步并行调用这些回调处理器。


## 基本用法
Tapable 使用时通常需要经历如下步骤：

- 创建钩子实例
- 调用订阅接口注册回调，包括：tap、tapAsync、tapPromise
- 调用发布接口触发回调，包括：call、callAsync、promise

```
const { SyncHook } = require("tapable");

// 1. 创建钩子实例
const sleep = new SyncHook();

// 2. 调用订阅接口注册回调
sleep.tap("test", () => {
  console.log("callback A");
});

// 3. 调用发布接口触发回调
sleep.call();

// 运行结果：
// callback A
```

## Tapable 钩子类型
Tabable 提供如下类型的钩子(统计数据来自 webpack@5.37.0)：


| 钩子   | 简介	|统计
|  ----                   | ----  | ---- |
|SyncHook	                | 同步钩子	      | Webpack 共出现 86 次，如 Compiler.hooks.compilation|
|SyncBailHook	            |同步熔断钩子      |	Webpack 共出现 90 次，如 Compiler.hooks.shouldEmit|
|SyncWaterfallHook	      |同步瀑布流钩子    |	Webpack 共出现 26 次，如 Compilation.hooks.assetPath|
|SyncLoopHook	            |同步循环钩子	     |Webpack 中未使用|
|AsyncParallelHook	      |异步并行钩子      |	Webpack 仅出现 6 次：Compiler.hooks.make|
|AsyncParallelBailHook	  |异步并行熔断钩子	  |Webpack 中未使用|
|AsyncSeriesHook	        |异步串行钩子      |	Webpack 共出现 32 次，如 Compiler.hooks.done|
|AsyncSeriesBailHook	    |异步串行熔断钩子   |	Webpack 共出现 9 次，如 Compilation.hooks.optimizeChunkModules|
|AsyncSeriesLoopHook      |	异步串行循环钩子  |	Webpack 中未使用|
|AsyncSeriesWaterfallHook	|异步串行瀑布流钩子  |	Webpack 共出现 3 次，如 ContextModuleFactory.hooks.beforeResolve|

Documentation: https://github.com/webpack/tapable

按回调逻辑，分为：
- 基本类型，名称不带 Waterfall/Bail/Loop 关键字，与通常 「订阅/回调」 模式相似，按钩子注册顺序，逐次调用回调
1. waterfall 类型：前一个回调的返回值会被带入下一个回调
2. bail 类型：逐次调用回调，若有任何一个回调返回非 undefined 值，则终止后续调用
3. loop 类型：逐次、循环调用，直到所有回调函数都返回 undefined
- 第二个维度，按执行回调的并行方式，分为：
1. sync ：同步执行，启动后会按次序逐个执行回调，支持 call/tap 调用语句
2. async ：异步执行，支持传入 callback 或 promise 风格的异步回调函数，支持 callAsync/tapAsync 、promise/tapPromise 两种调用语句


### SyncHook
```
const { SyncHook } = require("tapable");
let hook = new SyncHook()

hook.tap(fn) // 注册回调函数
hook.call()  // 按照顺序调用所有注册的回调函数
hook.callAsync(err => {}) //处理回调函数可能出现的错误
```

### SyncBailHook
bail 有保释的意思。若任一回调返回了非 undefined 的值，则中断后续处理，返回的值会作为call运行的结果

### SyncWaterfallHook
将前一个回调函数的返回值作为参数传入下一个函数。最后返回的值会作为call运行的结果

```
const { SyncWaterfallHook } = require("tapable");

class Somebody {
  constructor() {
    this.hooks = {
      sleep: new SyncWaterfallHook(["msg"]),
    };
  }
  sleep() {
    return this.hooks.sleep.call("hello");
  }
}

const person = new Somebody();

// 注册回调
person.hooks.sleep.tap("test", (arg) => {
  console.log(`call 调用传入： ${arg}`);
  return "tecvan";
});

person.hooks.sleep.tap("test", (arg) => {
  console.log(`A 回调返回： ${arg}`);
  return "world";
});

console.log("最终结果：" + person.sleep());
// 运行结果：
// call 调用传入： hello
// A 回调返回： tecvan
// 最终结果：world

```

- 初始化时必须提供参数，例如上例 new SyncWaterfallHook(["msg"]) 构造函数中必须传入参数 ["msg"] ，用于动态编译 call 的参数依赖，后面会讲到「动态编译」的细节。
- 发布调用 call 时，需要传入初始参数


## SyncLoopHook 
loop 型钩子的特点是循环执行直到所有回调都返回 undefined ，不过这里循环的维度是单个回调函数，例如有回调队列 [fn1, fn2, fn3] ，loop 钩子先执行 fn1 ，如果此时 fn1 返回了非 undefined 值，则继续执行 fn1 直到返回 undefined 后才向前推进执行 fn2

webpack没有使用到此hook

---

## 异步钩子
支持在回调函数中执行异步操作

### AsyncSeriesHook
- 支持异步回调，可以在回调函数中写 callback 或 promise 风格的异步操作
- 回调队列依次执行，前一个执行结束后才会开始执行下一个
- 与 SyncHook 一样，不关心回调的执行结果


使用tapAsync注册函数
```
//注册的回调函数里面的cb执行为之后，才会执行下一个函数
const { AsyncSeriesHook } = require("tapable");

const hook = new AsyncSeriesHook();

// 注册回调
hook.tapAsync("test", (cb) => {
  console.log("callback A");
  setTimeout(() => {
    console.log("callback A 异步操作结束");
    // 回调结束时，调用 cb 通知 tapable 当前回调已结束
    cb();
  }, 100);
});

hook.tapAsync("test", () => {
  console.log("callback B");
});

hook.callAsync();
// 运行结果：
// callback A
// callback A 异步操作结束
// callback B
```

使用promise注册
```
const { AsyncSeriesHook } = require("tapable");

const hook = new AsyncSeriesHook();

// 注册回调
hook.tapPromise("test", () => {
  console.log("callback A");
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log("callback A 异步操作结束");
      resolve();
    }, 100);
  });
});

hook.tapPromise("test", () => {
  console.log("callback B");
  return Promise.resolve();
});

hook.promise();
// 运行结果：
// callback A
// callback A 异步操作结束
// callback B
```

### AsyncParallelHook
是以并行方式，同时执行回调队列里面的所有回调


## 动态编译
Tapable 最大的秘密就是其内部实现了一套非常大胆的设计：  
动态编译，所谓的同步、异步、bail、waterfall、loop 等回调规则都是基于动态编译能力实现的，所以要深入学习 tapable 必然绕不开动态编译特性。