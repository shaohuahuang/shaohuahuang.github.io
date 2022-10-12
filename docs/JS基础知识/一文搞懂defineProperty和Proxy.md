## 一文搞懂Object.defineProperty和Proxy

### 前言
有的时候，我们在更改一个对象上的属性的时候，我们希望能够监听到这样的操作，并执行相关的逻辑。defineProperty合Proxy提供了这样的功能。今天我们一起来看看他们的用法

### Object.defineProperty
作用：在一个对象上定义新属性，或者修改现有属性，并返回这个对象

#### 基本使用
**语法**： `Object.defineProperty(obj, prop, descriptor)`

**参数**：
1. 需要添加属性的对象
2. 要定义或修改的属性名称或[Symbol]
3. 要定义或修改的属性描述符

我们来看一个简单的例子：

```javascript
let person = {}
let personName = 'Alice'

//在person上面添加属性name，值为personName
Object.defineProperty(person, 'name', {
  // 默认不可枚举 (for in打印不会打印出来), 要可枚举需要设置 enumerable: true
  // 默认可以修改
  // 默认不可以删除， 要可删除需要设置 configurable: true
  get: function(){
    console.log('触发get方法')
    return personName
  },
  set: function(val){
    console.log('触发set方法')
    personName = val
  }
})

//当读取person对象的name属性时，触发get方法
console.log(person.name)

//当修改personName时，重新访问person.namep发现修改成功
personName = 'bob'
console.log(person.name)

// 对person.name进行修改，触发set方法
person.name = 'bob'
console.log(person.name)
```

通过这种方法，我们成功监听了person上的name属性的变化。

### Proxy监听多个属性
defineProperty每次只能监听一个属性的变化，如果我们想要监听多个属性的话，可以使用Proxy。

**语法**：const p = new Proxy(target, handler)  

**参数**：  
1. target: 要使用Proxy包装的目标对象(可以是任何对象，包括原生数组，函数，甚至另一个代理) 
2. handler: 一个通常以函数作为属性的对象，各属性中的函数分别定义了在执行各种操作时代理p的行为

通过Proxy，我们可以对`设置代理的对象`上的一些操作进行拦截，外界对这个对象的各种操作，都要先通过这层拦截。（和defineProperty差不多）

先看一个简单的例子： 

```javascript
//定义一个需要代理的对象
let person = {
    age: 0,
    school: 'xdu'
}
//定义handler对象
let hander = {
    get(obj, key) {
        console.log('触发了get')
        // 如果对象里有这个属性，就返回属性值，如果没有，就返回默认值66
        return key in obj ? obj[key] : 66
    },
    set(obj, key, val) {
        console.log('触发了set')
        obj[key] = val
        return true
    }
}
//把handler对象传入Proxy
let proxyObj = new Proxy(person, hander)

// 测试get能否拦截成功
console.log(proxyObj.school)//输出：触发了get xdu
console.log(proxyObj.name)//输出：触发了get 66

// 测试set能否拦截成功
proxyObj.age = 18 // 输出：触发了set
console.log(proxyObj.age)//输出： 触发了get 18
```
> 值得注意的是:之前我们在使用Object.defineProperty()给对象添加一个属性之后，我们对对象属性的读写操作仍然在对象本身。
但是一旦使用Proxy，我们就要对Proxy的实例对象proxyObj进行操作。这是两者的一个区别

#### 监听数组上的操作
```javascript
let subject = ['高数']
let handler = {
    get(obj, key) {
        console.log('触发了get')
        return key in obj ? obj[key] : '没有这门学科'
    }, set(obj, key, val) {
         console.log('触发了set')
        obj[key] = val
        //set方法成功时应该返回true，否则会报错
        return true
    }
}

let proxyObj = new Proxy(subject, handler)

// 检验get和set
console.log(proxyObj)//输出：[ '高数' ]
console.log(proxyObj[1])//输出：触发了get 没有这门学科
proxyObj[0] = '大学物理' // 输出：触发了set
console.log(proxyObj[0])//输出：触发了get 大学物理

// // 检验push增加的元素能否被监听
proxyObj.push('线性代数') // 输出：触发get 触发get 触发set 触发set
console.log(proxyObj[1])//输出: 线性代数
```
> 为什么push时触发了两次get和两次set？这和push的实现原理有关：push操作除了增加数组的数据项之外，也会引发数组本身其他相关属性的改变


## Reference
https://juejin.cn/post/7069397770766909476