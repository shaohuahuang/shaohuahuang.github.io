# CSS动画

## 动作 transition 
transition 直译为过渡，即给属性变化添加过渡效果。
```
// 一个例子
.blueball {
  ...
  opacity: 1;
  transition: 1s;  /* 改变 opacity 属性，持续1秒 */
}
.blueball:hover {
  opacity: 0.3;
}
```

css里面的transition属性是一个简写的形式，实际上它由四个部分组成
```
transition-property: all; /* 过渡属性 */
transition-duration: 0; /* 耗时 */
transition-timing-function: ease; /* 效果，默认 ease（缓入缓出） */
transition-delay: 0; /* 延迟 */
```


## 动画 animation
transition 只能做单个动作，如果动画包含多个动作，这时候就需要 animation

我们先来看一个简单的动画  
![animation](https://pic3.zhimg.com/v2-5b95c4ae60bf66f047aaddba2a8d7f7a_b.webp)

这个动画明显由两个动作组成：蓝变绿，绿变橙。

两个连续的线段有三个关键点，两个连续的动作必然也有三个关键帧（keyframe），我们通过定义这三个关键帧（起点，蓝变绿，终点）来定义这两个动作。

CSS代码如下： 
```
.blueball {
  ...
  background-color: #0080ff; /* 蓝色 */
  position: relative;
  animation: forward 4s; /* 执行 forward 动画，耗时 4s */
}

/* 三个关键帧： 起点（蓝色），蓝变绿，终点（橙色） */
@keyframes forward {
  0% {left: 0; }
  50% {left: 200px; background-color: #009a61;}
  100% {left: 400px; background-color: orange;}
}
```

同样的，animation: forward 4s;也是简写形式，完整的 animation 属性包括（冒号后为默认值）：
```
animation-name: none; /* 动画名称 */
animation-duration: 0; /* 耗时 */
animation-timing-function: ease; /* 效果，默认缓入缓出 */
animation-delay: 0; /* 延迟 */
animation-iteration-count: 1; /* 循环次数 */
animation-direction: normal; /* 正放 or 倒放 */
```