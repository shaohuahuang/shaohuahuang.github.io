# Tree Shaking

顾名思义，tree shaking就是在编译阶段把程序里的无用代码删掉的过程。

看下面一个例子：
```
//main.js
import { post } from './util.js'

let baz = () => {
  post()
}

//util.js
export const post = () => { console.log('post') }
export const get = () => { console.log('get') }
```

util.js里的get函数没有被引用，编译的时候代码会被删除掉。

## Reference
[Tree-Shaking性能优化实践 - 原理篇](https://juejin.cn/post/6844903544756109319)
[你的Tree-Shaking并没什么卵用](https://zhuanlan.zhihu.com/p/32831172)