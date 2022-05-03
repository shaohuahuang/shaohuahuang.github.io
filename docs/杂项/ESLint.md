## ESLint

```
module.exports = {
    "extends": "airbnb" // extends让我们可以把其他的rule set包含进来
};
```

extends里面的内容：
1. ESLint的推荐配置 eg: eslint:recommended
2. 插件 eg: eslint:react/recommended ====> eslint-plugin-react
3. 三方包 eg: eslint-config-airbnb
4. 本地路径 eg: ./rules/react


## AST
ESLint是使用[espree](https://github.com/eslint/espree) 来解析JS语句

就像 CSS 选择器一样，AST 选择器也有多种规则让我们可以更方便的选中特定的代码片段，具体规则可以参考

[Selectors - ESLint - Pluggable JavaScript linter](https://eslint.org/docs/developer-guide/selectors%23what-syntax-can-selectors-have)
[GitHub - estools/esquery: ECMAScript AST query library](https://github.com/estools/esquery)


## 单条 rule 是怎么工作的？
关于如何写一条 rule，官方文档中 (Working with Rules)[https://eslint.org/docs/developer-guide/working-with-rules] 一节中已经有了详细的阐述，这里只做简单的描述。

