## Time to Interactive (TTI)

### 定义
页面从开始加载到可以稳定的响应用户操作的所需的时间

### 计算方法
step1:
从FCP开始，找到第一个至少5s的安静窗口(quiet window)
> 安静窗口: 没有longs tasks(占用主线程超过50ms)，并且期间发出不超过两次GET请求

step2: 找到安静窗口前的最后一个long task。 如果不存在，那么FCP = TTI

step3: TTI = long task结束的时间

![](./images/TTI.svg)

### 怎么测量？
TTI应该在发布功能之前测量，因为用户的行为可能会导致TTI不准。我们可以使用Lighthouse来检查我们的站点。
> 为了达到好的用户体验，一般TTI的时间要控制在5s以内。

### 怎么提高TTI 

- Minify JavaScript
- Preconnect to required origins
- Preload key requests
- Reduce the impact of third-party code
- Minimize critical request depth
- Reduce JavaScript execution time
- Minimize main thread work
- Keep request counts low and transfer sizes small


## Reference
https://web.dev/tti/  
https://juejin.cn/post/7128400578467594248