# JS模块化

## CommonJS
CommonJS规范是由NodeJS发扬光大, 标志着JavaScript模块化编程正式登上舞台。


- 定义模块  
根据CommonJS规范，一个单独的文件就是一个模块。每一个模块都是一个单独的作用域，也就是说，在该模块内部定义的变量，无法被其他模块读取，除非定义为global对象的属性

- 模块输出：  
模块只有一个出口，module.exports对象，我们需要把模块希望输出的内容放入该对象

- 加载模块：  
加载模块使用require方法，该方法读取一个文件并执行，返回文件内部的module.exports对象

## ES6 Module
对于ES6来说，不必再使用闭包和封装函数等方式进行模块化支持了。在ES6中，从语法层面就提供了模块化的功能。然而受限于浏览器的实现程度，如果想要在浏览器中运行，还是需要通过Babel等转译工具进行编译。

#### 特点：  
- 按需加载（编译时加载）  
- import和export命令只能在模块的顶层，不能在代码块之中（如：if语句中）,import()语句可以在代码块中实现异步动态按需动态加载

#### 语法：
- 导入：import {模块名A，模块名B...} from '模块路径'
- 导出：export和export default
- import('模块路径').then()方法

注意：<span style="color: red">export只支持对象形式导出，不支持值的导出，export default命令用于指定模块的默认输出，只支持值导出，但是只能指定一个，本质上它就是输出一个叫做default的变量或方法。</span>

```
/*错误的写法*/
// 写法一
export 1;

// 写法二
const m = 1;
export m;

// 写法三
if (x === 2) {
  import MyModual from './myModual';
}

/*正确的三种写法*/
// 写法一
export const m = 1;

// 写法二
cosnt m = 1;
export {m};

// 写法三
const n = 1;
export {n as m};

// 写法四
const n = 1;
export default n;

// 写法五
if (true) {
    import('./myModule.js')
    .then(({export1, export2}) => {
      // ...·
    });
}
// 写法六
Promise.all([
  import('./module1.js'),
  import('./module2.js'),
  import('./module3.js'),
])
.then(([module1, module2, module3]) => {
   ···
});
```

## CommonJS vs ES6 Module
#### 区别：

1. CommonJs模块输出的是<b style="color:red">值的拷贝</b>，也就是说，一旦输出一个值，模块内部的变化不会影响到这个值。

```
// common.js
let count = 1;
const addCount = () => {
    return ++count;
}
module.exports = {
    addCount,
    count
};
 // index.js
const res = require('./common.js');
console.log(res.count) // 1
res.addCount();
console.log(res.count) // 1
```


<span style="color: red">ES6 module静态分析，动态引用，输出的是值的引用，值改变，引用也改变，即原来模块中的值改变则该加载的值也改变</span>

```
// es6.js
let count = 1;
const addCount = () => {
    return ++count;
}
export {
    addCount,
    count
}
// main.js
import  { count, addCount } from './es6';
console.log(count) // 1
addCount();
console.log(count) // 2
```


2. commonJS引用基础数据类型是复制，对于复杂数据类型是浅拷贝，可以重新赋值

```
// common.js
let count = 1;
const addCount = () => {
    return ++count;
}
module.exports = {
    addCount,
    count
};

// index.js
let {count} = require('./common.js');
console.log(count) // 1
count = 4
console.log(count) // 4
```

而 ES6 module 引用的值<span style="color: red">只读，不可修改</span>

```
// es6.js
let count = 1;
const addCount = () => {
    return ++count;
}
export default {
    addCount,
    count
}
// main.js

import { count, addCount } from './es6'
console.log(count)
// addCount();
count = 4 // 报错 
console.log(count)
```
