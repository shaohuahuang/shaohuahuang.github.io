## preload

### 前言
preload 是一个新的 Web 标准，旨在提高性能以及提供给 web 开发者更多的细粒度加载控制。

preload 可以指明哪些资源在页面加载的生命周期的早期阶段就开始获取。这一机制使得资源可以更早的得到加载并可用，且更不易阻塞页面的初步渲染，进而提升性能。

可以提早获取的资源：
- css里面用到的文件：字体和图片
- JS可能会获取的资源，譬如JSON, 动态导入的脚本，webworker
- 大的图片和video

### 哪些类型的资源可以使用 preload？
- vaudio: 音频文件。
- document: 一个将要被嵌入到 frame 或 iframe 内部的 html 文档。
- embed: 一个将要被嵌入到 <embed> 元素内部的资源。
- fetch: 那些将要通过 fetch 和 XHR 请求来获取的资源，比如一个 ArrayBuffer 或 jsON 文件。
- font: 字体文件。
- image: 图片文件。
- object: 一个将会被嵌入到 embed 元素内的文件。
- script: JavaScript 文件。
- style: 样式表。
- track: WebVTT 文件。
- worker : 一个 JavaScript 的 web worker 或 shared worker。
- video: 视频文件。

### 如何使用 preload？
```
<head>
  <meta charset="utf-8" />
  <title>JS and CSS preload example</title>

  <link rel="preload" href="style.css" as="style" />
  <link rel="preload" href="main.js" as="script" />

  <link rel="stylesheet" href="style.css" />
</head>

<body>
  <h1>bouncing balls</h1>
  <canvas></canvas>

  <script src="main.js" defer></script>
</body>
```

####  使用 as 来指定将要预加载的内容的类型，将使得浏览器能够：  

- 更精确地优化资源加载优先级。
- 匹配未来的加载需求，在适当的情况下，重复利用同一资源。
- 为资源应用正确的内容安全策略。
- 为资源设置正确的 Accept 请求头。

### 更早的加载
使用 preload 前，在遇到资源依赖时才会进行加载：

使用 preload 后，浏览器会进行资源调度，将 link 指定的资源优先加载，不管资源是否使用; 
> 譬如一个图片的下载本来是要等到前面的JS脚本执行完成之后，DOM完成解析之后，才能开始下载。有了preload之后，图片可以在JS下载完成之后就可以进行下载了。

### 不要混淆 preload 和 prefetch
```
<!-- 预测 -->
<link rel="preload" href="a.css">
<!-- 确认 --> 
<link rel="prefetch" href="b.css">
```
prefetch 是一种**期望**，预测会加载指定的资源，**以备下一个导航或者下一屏页面使用**，但**对当前的页面并没有什么帮助**。如果 prefetch 使用不得当，还会造成资源重复加载的问题。页面不一定会使用 prefetch 指定的资源。
> prefetch需要慎重使用。通常是在动态的情况下进行使用，譬如搜索的时候，用户可能会点击前面几个搜索结果，所以可以对前几个搜索结果对应的链接进行prefetch，从而让用户可以流畅的打开下一页。

preload 是一种**肯定**，确认会加载指定资源，在页面加载的生命周期的早期阶段就开始获取，不区分下一屏。页面一定会使用 preload 指定的资源（不使用将会报警告）。

### 总结
- 资源优先级 Priority 会随着浏览器进程的不同阶段动态变化的。
- 更早的加载意味着更早的渲染。
- prefetch 和 preconnect 都属于预加载机制，但 preload 在大多数情况下更符合需求和预期。

## Reference 
- https://www.cnblogs.com/xiaonian8/p/13663770.html
- [MDN preload 文档](https://developer.mozilla.org/en-US/docs/Web/HTML/Link_types/preload)