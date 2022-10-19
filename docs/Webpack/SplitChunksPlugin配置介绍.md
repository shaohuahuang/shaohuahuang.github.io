## SplitChunksPlugin配置介绍

### 前言
splitChunks插件可以用来做webpack的分包。当代码不需要在首屏加载的时候，我们可以把这部分代码单独打包到chunk中。 

### 默认配置
```javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async',
      minSize: 20000,
      minRemainingSize: 0,
      minChunks: 1,
      maxAsyncRequests: 30,
      maxInitialRequests: 30,
      enforceSizeThreshold: 50000,
      cacheGroups: {
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          reuseExistingChunk: true,
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
      },
    },
  },
};
```

### 配置选项字段解释
- chunks选项，决定要提取那些模块。

  - 默认是async：只提取异步加载的模块出来打包到一个文件中。

    异步加载的模块：通过import('xxx')或require(['xxx'],() =>{})加载的模块。


  - initial：提取同步加载和异步加载模块，如果xxx在项目中异步加载了，也同步加载了，那么xxx这个模块会被提取两次，分别打包到不同的文件中。

  同步加载的模块：通过 import xxx或require('xxx')加载的模块。


  - all：不管异步加载还是同步加载的模块都提取出来，打包到一个文件中。


- minSize选项：规定被提取的模块在压缩前的大小最小值，单位为字节，默认为30000，只有超过了30KB才会被提取。
- maxSize选项：把提取出来的模块打包生成的文件大小不能超过maxSize值，如果超过了，要对其进行分割并打包生成新的文件。单位为字节，默认为0，表示不限制大小。
- minChunks选项：表示要被提取的模块最小被引用次数，引用次数超过或等于minChunks值，才能被提取。
- maxAsyncRequests选项：最大的按需(异步)加载次数，默认为 6。
- maxInitialRequests选项：打包后的入口文件加载时，还能同时加载js文件的数量（包括入口文件），默认为4。
先说一下优先级 maxInitialRequests / maxAsyncRequests <maxSize <minSize。
- automaticNameDelimiter选项：打包生成的js文件名的分割符，默认为~。
- name选项：打包生成js文件的名称。
- cacheGroups选项，**核心重点**，配置提取模块的方案。里面每一项代表一个提取模块的方案。下面是cacheGroups每项中特有的选项，其余选项和外面一致，若cacheGroups每项中有，就按配置的，没有就使用外面配置的。

  - test选项：用来匹配要提取的模块的资源路径或名称。值是正则或函数。
  - priority选项：方案的优先级，值越大表示提取模块时优先采用此方案。默认值为0。
  - reuseExistingChunk选项：true/false。为true时，如果当前要提取的模块，在已经在打包生成的js文件中存在，则将重用该模块，而不是把当前要提取的模块打包生成新的js文件。
  - enforce选项：true/false。为true时，忽略minSize，minChunks，maxAsyncRequests和maxInitialRequests外面配置选项。

## Reference
https://juejin.cn/post/6844904198023168013#heading-5