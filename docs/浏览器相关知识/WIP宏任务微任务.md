
宏任务与微任务的几种创建方式 👇

|  宏任务（Macrotask）   |    微任务（Microtask）  |
|  ----  | ----  |
| setTimeout  |    requestAnimationFrame（有争议）|
| setInterval  | MutationObserver（浏览器环境） |
| MessageChannel  | Promise.[ then/catch/finally ] |
| I/O，事件队列  | process.nextTick（Node环境） |
| setImmediate（Node环境）  | queueMicrotask |
| script（整体代码块）  |  |


如何理解 script（整体代码块）是个宏任务呢 🤔

实际上如果同时存在两个 script 代码块，会首先在执行第一个 script 代码块中的同步代码，如果这个过程中创建了微任务并进入了微任务队列，第一个 script 同步代码执行完之后，会首先去清空微任务队列，再去开启第二个 script 代码块的执行。所以这里应该就可以理解 script（整体代码块）为什么会是宏任务。