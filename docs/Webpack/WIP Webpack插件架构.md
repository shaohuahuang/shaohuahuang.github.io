# Webpack插件架构: Tapable框架

webpack 的插件体系是一种基于 Tapable 实现的强耦合架构，它在特定时机触发钩子时会附带上足够的上下文信息，插件定义的钩子回调中，能也只能与这些上下文背后的数据结构、接口交互产生 side effect，进而影响到编译状态和后续流程。

## 插件的基本形态
```
class SomePlugin {
    apply(compiler) {
    }
}
```

Webpack 会在启动后按照注册的顺序逐次调用插件对象的 apply 函数，同时传入编译器对象 compiler ，插件开发者可以以此为起点触达到 webpack 内部定义的任意钩子，例如：
```
class SomePlugin {
    apply(compiler) {
        compiler.hooks.thisCompilation.tap('SomePlugin', (compilation) => {
        })
    }
}
```
> Tapable设计可参照上一篇[Tapable介绍](./Tapable.md)


## Webpack 插件架构
一个好的插件架构需要解决以下三个问题：

-「接口」：需要提供一套逻辑接入方法，让开发者能够将逻辑在特定时机插入特定位置
-「输入」：如何将上下文信息高效传导给插件
-「输出」：插件内部通过何种方式影响整套运行体系


针对这些问题，webpack 为开发者提供了基于 tapable 钩子的插件方案：

- 编译过程的特定节点以钩子形式，通知插件此刻正在发生什么事情；
- 通过 tapable 提供的回调机制，以参数方式传递上下文信息；
- 在上下文参数对象中附带了很多存在 side effect 的交互接口，插件可以通过这些接口改变

钩子回调传递的 compilation/params 参数就是 webpack 希望传递给插件的上下文信息，也是插件能拿到的输入。不同钩子会传递不同的上下文对象，这一点在钩子被创建的时候就定下来了，比如：
```
class Compiler {
    constructor() {
        this.hooks = {
            /** @type {SyncBailHook<Compilation>} */
            shouldEmit: new SyncBailHook(["compilation"]),
            /** @type {AsyncSeriesHook<Stats>} */
            done: new AsyncSeriesHook(["stats"]),
            /** @type {AsyncSeriesHook<>} */
            additionalPass: new AsyncSeriesHook([]),
            /** @type {AsyncSeriesHook<Compiler>} */
            beforeRun: new AsyncSeriesHook(["compiler"]),
            /** @type {AsyncSeriesHook<Compiler>} */
            run: new AsyncSeriesHook(["compiler"]),
            /** @type {AsyncSeriesHook<Compilation>} */
            emit: new AsyncSeriesHook(["compilation"]),
            /** @type {AsyncSeriesHook<string, Buffer>} */
            assetEmitted: new AsyncSeriesHook(["file", "content"]),
            /** @type {AsyncSeriesHook<Compilation>} */
            afterEmit: new AsyncSeriesHook(["compilation"]),
        };
    }
}
```

常见的参数对象有 compilation/module/stats/compiler/file/chunks 等，在钩子回调中可以通过改变这些对象的状态，影响 webpack 的编译逻辑


## 名词解释
- Entry：编译入口，webpack 编译的起点
- Compiler：编译管理器，webpack 启动后会创建 compiler 对象，该对象一直存活知道结束退出
- Compilation：单次编辑过程的管理器，比如 watch = true 时，运行过程中只有一个 compiler 但每次文件变更触发重新编译时，都会创建一个新的 compilation 对象
- Dependence：依赖对象，webpack 基于该类型记录模块间依赖关系
- Module：webpack 内部所有资源都会以“module”对象形式存在，所有关于资源的操作、转译、合并都是以 “module” 为基本单位进行的
- Chunk：编译完成准备输出时，webpack 会将 module 按特定的规则组织成一个一个的 chunk，这些 chunk 某种程度上跟最终输出一一对应
- Loader：资源内容转换器，其实就是实现从内容 A 转换 B 的转换器
- Plugin：webpack构建过程中，会在特定的时机广播对应的事件，插件监听这些事件，在特定时间点介入编译过程



## Webpack编译的几个阶段

1. compiler.make 阶段：
- entry 文件以 dependence 对象形式加入 compilation 的依赖列表，dependence 对象记录有 entry 的类型、路径等信息
- 根据 dependence 调用对应的工厂函数创建 module 对象，之后读入 module 对应的文件内容，调用 loader-runner 对内容做转化，转化结果若有其它依赖则继续读入依赖资源，重复此过程直到所有依赖均被转化为 module
2. compilation.seal 阶段：
- 遍历 module 集合，根据 entry 配置及引入资源的方式，将 module 分配到不同的 chunk
- 遍历 chunk 集合，调用 compilation.emitAsset 方法标记 chunk 的输出规则，即转化为 assets 集合
3. compiler.emitAssets 阶段：
将 assets 写入文件系统


## 什么时候会触发钩子

- compiler.hooks.compilation ：
时机：启动编译创建出 compilation 对象后触发
参数：当前编译的 compilation 对象
示例：很多插件基于此事件获取 compilation 实例

- compiler.hooks.make：
时机：正式开始编译时触发
参数：同样是当前编译的 compilation 对象
示例：webpack 内置的 EntryPlugin 基于此钩子实现 entry 模块的初始化

- compilation.hooks.optimizeChunks ：
时机： seal 函数中，chunk 集合构建完毕后触发
参数：chunks 集合与 chunkGroups 集合
示例： SplitChunksPlugin 插件基于此钩子实现 chunk 拆分优化

- compiler.hooks.done：
时机：编译完成后触发
参数： stats 对象，包含编译过程中的各类统计信息
示例： webpack-bundle-analyzer 插件基于此钩子实现打包分析












## Reference 
[Webpack 插件架构深度讲解](https://zhuanlan.zhihu.com/p/367931462)
[一文吃透 Webpack 核心原理](https://zhuanlan.zhihu.com/p/363928061)