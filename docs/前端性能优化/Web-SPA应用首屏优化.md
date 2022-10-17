## Web SPA 应用首屏加载优化

### 动态加载辅助资源
一些不需要再首页使用的外部脚本，可以在运行时通过创建script标签，动态加载 

### 页面路由按需加载
我们可以通过Suspense，lazy来达到路由懒加载的目的。

```javascript
import { Suspense, lazy } from 'react'

const DemoA = lazy(() => import('./demo/a'))
const DemoB = lazy(() => import('./demo/b'))

<Suspense>
  <NavLink to="/demoA">DemoA</NavLink>
  <NavLink to="/demoB">DemoB</NavLink>

  <Router>
    <DemoA path="/demoA" />
    <DemoB path="/demoB" />
  </Router>
</Suspense>
```

#### 代码分割
我们可以对 node_modules 第三方依赖 打包资源拆分细化成多个资源文件，借助浏览器支持 HTTP 同时发起多个请求特性，使得资源异步并行加载，从而提高资源加载速度。

```
// webpack.config.js
module.exports = function ({ production = false, development = false }) {
  ...
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'static/js/[name].[contenthash:8].js', // 主文件分割出的文件命名
    chunkFilename: 'static/js/[name].[contenthash:8].chunk.js', // splitChunks 分割出的文件命名
  },
  
  optimization: {
      minimize: true,
      ...
      splitChunks: {
        chunks: 'all',
        cacheGroups: {
          default: false,
          vendors: false,
          // 多页面应用，或者 webpack import() 多个 chunk 文件中，有 import 其他模块两次或者多次时，会打包生成 common
          common: {
            chunks: "all",
            minChunks: 2,
            name: 'common',
            enforce: true,
            priority: 5
          }, 
          // node_modules 中的公共模块抽离
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            chunks: 'initial',
            enforce: true,
            priority: 10,
            name: 'vendor'
          },
          // @materual-ui
          material: {
            name: 'chunk-material',
            priority: 20, // 优先级高于 vendor
            test: /[\\/]node_modules[\\/]_?@material-ui(.*)/
          },
        }
      },
      runtimeChunk: { // 运行时代码（webpack执行时所需的代码）从主文件中抽离
        name: entrypoint => `runtime-${entrypoint.name}`,
      },
    },
})
```

#### 静态图片资源采用 cdn 方式
使用CDN可以让用户从最近的节点拉取资源，减少网络请求时间。