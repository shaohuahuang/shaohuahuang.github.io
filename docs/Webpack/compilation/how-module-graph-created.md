## webpack里模块的依赖图是如何构建的

### 前言
上一篇文章我们分析了webpack是如何把入口js文件的依赖找出来的，这次我们来来讲讲webpack事如何把整个项目的依赖关系图建好的

### 概念解释
- ModuleGraph: 用来表示依赖关系的类

### 解析过程
在上一篇文章里面，我们讲到了buildModule，buildModule处理完之后，会执行processModuleDependencies
这个函数会把已经完成解析的moudle放到processDependenciesQueue里面，来分析模块的依赖
```
this.processDependenciesQueue = new AsyncQueue({
  name: "processDependencies",
  parallelism: options.parallelism || 100,
  processor: this._processModuleDependencies.bind(this)
});
```

这里的processor函数是_processModuleDependencies, 我们看啊他里面的执行逻辑
```
/** @type {DependenciesBlock[]} */
const queue = [module];
do {
  const block = queue.pop();
  if (block.dependencies) {
    currentBlock = block;
    let i = 0;
    for (const dep of block.dependencies) processDependency(dep, i++);
  }
  if (block.blocks) {
    for (const b of block.blocks) queue.push(b);
  }
} while (queue.length !== 0);

if (--inProgressSorting === 0) onDependenciesSorted();
```

对于一个module，对他的每个依赖，再走一遍factorize -> add -> build的过程。这样子就能最终把依赖树构建出来。之后我们打包的时候，只需要对依赖树进行BFS遍历，把每个节点的source聚合在一起，就可以输出打包文件了
