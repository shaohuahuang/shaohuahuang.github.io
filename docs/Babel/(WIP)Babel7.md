# Babel 7 

## 核心库 @babel/core 

## 命令行工具 @babel/cli
```
"scripts": {
    "compiler": "babel src --out-dir lib --watch" //命令行编译JS文件
}
```

## 插件
Babel 构建在插件之上，使用现有的或者自己编写的插件可以组成一个转换通道，Babel 的插件分为两种: 语法插件和转换插件。

### 语法插件
这些插件只允许 Babel 解析（parse） 特定类型的语法（不是转换），可以在 AST 转换时使用，以支持解析新语法，例如：
```
import * as babel from "@babel/core";
const code = babel.transformFromAstSync(ast, {      
  plugins: ["@babel/plugin-proposal-optional-chaining"],  //支持可选链 
  babelrc: false}).code;
```

### 转换插件
转换插件会启用相应的语法插件，通过构建AST进行转换

### 插件的使用
```
//.babelrc{    
  "plugins": ["@babel/plugin-transform-arrow-functions"]
}
```

### preset
插件多了，配置起来会非常繁琐。我们可以通过preset来轻松使用一组插件

#### 官方Preset
- @babel/preset-env
- @babel/preset-flow
- @babel/preset-react
- @babel/preset-typescript
> 从 Babel v7 开始，所有针对标准提案阶段的功能所编写的预设(stage preset)都已被弃用，官方已经移除了 @babel/preset-stage-x

**@babel/preset-env**  
@babel/preset-env 主要作用是对我们所使用的并且目标浏览器中缺失的功能进行代码转换和加载 polyfill
```
//.babelrc{    
  "presets": ["@babel/preset-env"]
}
```

**@babel/polyfill**  
@babel/polyfill 模块包括 core-js 和一个自定义的 regenerator runtime 模块，可以模拟完整的 ES2015+ 环境
>V7.4.0之后，已经废弃

**@babel/plugin-transform-runtime**  
@babel/plugin-transform-runtime 是一个可以重复使用 Babel 注入的帮助程序，以节省代码大小的插件。例如所有的包含class的文件，再不使用插件的时候，都需要注入_classCallCheck、_defineProperties、_createClass这几个方法的代码。
```
npm install --save-dev @babel/plugin-transform-runtime
npm install --save @babel/runtime
```
使用插件之后，方法可以从@babel/runtime里面导出，从而避免重复使用

