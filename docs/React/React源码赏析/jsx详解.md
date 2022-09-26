## jsx详解

### 前言  
最近开始看React的源码。首先让我感到好奇的就是ReactDOM.render函数, 我们再入口文件里面通常会这样调用 `ReactDOM.render(<App />, document.getElememntById('root'));`
这个函数对应的的签名是
`render(element: React$Element, container, callback?)`
这里的React$Element应该是和JSX相关的，但是它具体是长什么样子了，我觉得值得写一篇文章来分析一下

   
### Babel对JSX语法的转化
当我们在写React组件的时候，Babel会对JSX语法进行转化。我们先来看几个简单的例子，看看Babel把jsx转化成什么了。

1. 简单的JSX

```javascript
const Test1 = () => {
 return <div>Feiwu</div> 
}

//-------转化之后--------------------------
const Test1 = () => {
  return React.createElement("div", null, "Feiwu");
};

```

这里我们可以看到，Babel把jsx转译成React.createElement函数了。createElement可接受多个参数。
前两个是type, props，后面的参数都是当前组件的children 

2. 带有useState数据的JSX

```javascript
const Test1 = () => {
  const [count, setCount] = useState(0)
  return <div onClick={
   () => setCount(count + 1)
    }>
    <p>Count</p>{count}
 </div> 
}

//-------转化之后--------------------------

const Test1 = () => {
  const [count, setCount] = useState(0);
  return React.createElement(
      "div", 
      {
        onClick: () => setCount(count + 1)
      }, 
      React.createElement("p", null, "Count"), 
      count
  );
};
```

这里我们可以发现，div上的onClick函数被放入到props对象里面。p元素和{count}都作为children参数传入createElement中

3. Nested JSX

```javascript
const Test1 = () => {
  const [count, setCount] = useState(0)
  return <div onClick={
   () => setCount(count + 1)
    }>
    <p>Count</p> {count}
 </div> 
}


const Test2 = () => {
    return <div>
      <Test1 />
      <p>Test2</p>
    </div> 
}

//-----------------------------------------
const Test1 = () => {
  const [count, setCount] = useState(0);
  return React.createElement("div", {
    onClick: () => setCount(count + 1)
  }, React.createElement("p", null, "Count"), " ", count);
};

const Test2 = () => {
  return React.createElement(
      "div", 
      null, 
      React.createElement(Test1, null), 
      React.createElement("p", null, "Test2"));
};
```

这里我们看到Test2引用了Test1这个函数组件，此时Test2的第一个child是React.createElement(Test1, null)。Test1被作为参数传入进来，我们接下里会来具体的看createElement的运行逻辑 


### createElement逻辑
```javascript
export function createElement(type, config, children) {
  let propName;

  // Reserved names are extracted
  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  if (config != null) {
    key = config.key
    ref = config.ref
    self = config.__self || undefined
    source = config.__source || undefined 
    
    // Remaining properties are added to a new props object
    // 上面这四个值是reserver props，不会放入到props对象里面。其他的属性都会放入props对象 
  }

  // 把children放入到props对象中
  props.children = children

  // 处理type.defaultProps，将其也放入到props对象

  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```
这段代码是简化后的createElement的逻辑，做的事情非常简单
- 把key, ref, __self, __source这四个reserved props单独拎出来
- 将config中的其他属性放入到props对象中
- 把children数组赋值给props.children
- 调用ReactElement，并返回结果


### ReactElement
```javascript
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // This tag allows us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  };

  return element;
};
```

## 总结
把createElement和ReactElement这两个函数的源码梳理清楚，我们就能知道React$Element这个类型是下面这个样子
```
{
    $$typeof: REACT_ELEMENT_TYPE,
    type: type,
    key: key,
    ref: ref,
    props: props,
    _owner: owner,
}
```
这是一个多层嵌套的数据结构，props里的children也是一样的格式。
