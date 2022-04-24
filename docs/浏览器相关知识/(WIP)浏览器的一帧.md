# 浏览器的一帧里面会处理哪些事情？ 

一般浏览器的刷新率为60HZ，即1秒钟刷新60次。1000ms / 60hz = 16.6 ，大概每过16.6ms浏览器会渲染一帧画面。

在这段时间内，浏览器大体会做两件事：task与render。

- task被称为宏任务，包括 setTimeout，setInterval，setImmediate，postMessage，requestAnimationFrame，I/O，DOM 事件 等。  
- render是指渲染页面。

## requestAnimationFrame
task没有办法精准的控制执行时机。  
requestAnimationFrame可以在每一帧render前被调用。  
一般被用来绘制动画，因为当代码执行完成之后就进入render，动画效果可以最快被呈现。

## requestIdleCallback
渲染完成后还有空闲时间则调用

如果task执行时间超过了16.6ms，那么就会出现掉帧的情况

React15中，采用递归的不可中断方式构建虚拟DOM树。  

为了解决掉帧造成的卡顿，React16将递归的构建方式改为可中断的遍历。React16就是基于requestIdleCallbackAPI，实现了自己的Fiber Reconciler。

以5ms的执行时间划分task，每遍历完一个节点，就检查当前task是否已经执行了5ms。

如果超过5ms，则中断本次task。

### requestIdleCallback 的缺陷
> requestIdleCallback is called only 20 times per second - Chrome on my 6x2 core Linux machine, it's not really useful for UI work




