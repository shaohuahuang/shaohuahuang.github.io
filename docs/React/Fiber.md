# Fiber

要解决的问题：
React 15里的Stack Reconciler是通过递归来找到需要更新的节点，无法终止。

解决方法：
React 16.0 Fiber Reconciler
1. 把更新和渲染任务分割成小的任务，逐步执行


Questions:
1. Fiber Tree的双缓冲优化策略 https://www.jianshu.com/p/6660d3ab0394  
2. React的requestIdleCallback是如何自己实现的  
https://zhuanlan.zhihu.com/p/60189423
https://blog.csdn.net/LuckyWinty/article/details/121154921

## Reference
https://medium.com/imgcook/a-closer-look-at-react-fiber-b2ab072fcc2a
https://blog.csdn.net/qq_32247819/article/details/119716284
https://zhuanlan.zhihu.com/p/372809882