## type和interface的异同

### 前言
在Typescript里面，有两个概念十分容易混淆，那便是 type 和 interface，它俩都可以用来表示 **接口**，但是实际使用上会存在一些差异，今天我们聊聊它俩，彻底弄清它俩的联系与区别。

### type和interface的相同点
1. 两者是对 接口定义 的两种不同形式，目的都是一样的，都是用来定义 **对象** 或者 **函数** 的形状，示例如下
```javascript
 interface example {
    name: string
    age: number
}
interface exampleFunc {
    (name:string,age:number): void
}


type example = {
    name: string
    age: number
}
type example = (name:string,age:number) => void
```

2. 它俩也支持 继承，只是具体的形式稍有差别。 如下所示：
```javascript
type exampleType1 = {
    name: string
}
interface exampleInterface1 {
    name: string
}

type exampleType2 = exampleType1 & {
    age: number
}
type exampleType2 = exampleInterface1 & {
    age: number
}

interface exampleInterface2 extends exampleType1 {
    age: number
}
interface exampleInterface2 extends exampleInterface1 {
    age: number
}
```

3. 

可以看到对于interface来说，继承是通过extends来实现的，而type是通过&来实现的。





### type和interface的不同点
首先聊聊type可以做到，但interface不能做到的事情

type可以定义 基本类型的别名，如 `type myString = string`
type可以通过 typeof 操作符来定义，如 `type myType = typeof someObj`
type可以申明 联合类型，如 `type unionType = myType1 | myType2`
type可以申明 元组类型，如 `type yuanzu = [myType1, myType2]`

接下来聊聊interface可以做到，但是type不可以做到的事情
1. type可以用来定义基本类型，而interface仅限于描述对象类型

2. interface可以 声明合并，示例如下
```javascript
interface test {
    name: string
}
interface test {
    age: number
}
// test实际为 { name: string; age: number }
```


> 这种情况下，如果是type的话，就会报 重复定义 的警告，因此是无法实现 声明合并 的

## Reference
https://juejin.cn/post/7059725643365220366