## 一个js文件的依赖是如何被Webpack解析出来的?

### 前言
最近在仔细的研究Webpack源码，因为是第一次研究，所以希望从最基本的开始。  
我们都知道webpack打包的流程是根据入口js文件，通过BFS遍历分析出所有的依赖关系，然后通过loader转化代买，最后打包并聚合成最终的bundle文件。  
这篇文章想要结合源码分析，webpack是如何根据入口文件，找出他的依赖的。
这个乍看起来似乎很简单，但是由于webpack的插件太多了，源码阅读起来非常困难，我也是花了九牛二虎之力，才理解了整个过程。


### 概念解释
- compiler: 编译器，Compiler的实例，**全局唯一**，定义了webpack整个构建流程。其提供了一系列钩子, 允许plugin在不同的时机介入 [Compiler Hooks](https://webpack.js.org/api/compiler-hooks/)
- compilation: 单次编译的管理实例，譬如在watch模式下，每次文件改动触发的编译都会创建该实例 [Compilation Hooks](https://webpack.js.org/api/compilation-hooks/)
- NormalModuleFactory: 用来生成module的类，里面也定义了一系列[hooks](https://webpack.js.org/api/normalmodulefactory-hooks/)
- JavascriptParser: javascript的parser，除了把源码转成ast之外，里面也定义了一系列[hooks](https://webpack.js.org/api/parser/)


### 解析过程
我们整理跳过前期的webpack准备工作，直接从compilation.addEntry这个函数开始，一步步看webpack是如何工作的。

addEntry -> _addEntryItem -> addModuleTree
```
addModuleTree({ context, dependency, contextInfo }, callback) {
  //...
  const Dep = /** @type {DepConstructor} */ (dependency.constructor);
  const moduleFactory = this.dependencyFactories.get(Dep);
  //...

  this.handleModuleCreation(
    {
      factory: moduleFactory,
      dependencies: [dependency],
      originModule: null,
      contextInfo,
      context
    },
    //...
  );
}
```

**问题**：这里的moduleFactoy是什么，他是什么时候被放入this.dependencyFactories里面的？
通过全局搜索，我们可以发现在EntryPlugin的apply方法，里面有操作这个字段的逻辑，如下所示
```
apply(compiler) {
  compiler.hooks.compilation.tap(
    "EntryPlugin",
    (compilation, { normalModuleFactory }) => {
      compilation.dependencyFactories.set(
        EntryDependency,
        normalModuleFactory
      );
    }
  );

  const { entry, options, context } = this;
  const dep = EntryPlugin.createDependency(entry, options);

  compiler.hooks.make.tapAsync("EntryPlugin", (compilation, callback) => {
    compilation.addEntry(context, dep, options, err => {
      callback(err);
    });
  });
}
```
由此可见moudleFactory其实就是指的normalMoudleFactory. 

我们继续看接下来的流程
addModuleTree -> handleModuleCreation -> factorizeModule
这里会把入口文件塞入到factorizeQueue里面
```
this.factorizeQueue = new AsyncQueue({
    name: "factorize",
    parent: this.addModuleQueue,
    processor: this._factorizeModule.bind(this)
});
```
factorizeQueue里面定义了一个processor，这个prorcessor每次有元素加到queue里面都会执行
所以我们接下来来看_factorizeModule里面的逻辑








### Reference

- Module相关的UML类图
![Module相关的UML类图](../images/module-uml.jpg)
