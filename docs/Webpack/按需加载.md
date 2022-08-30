## 按需加载

要实现按需加载，我们首先想到的是使用动态创建script标签的方法，来下载脚本并执行。但是在实现的过程中，有以下几个问题需要考虑：

1. 如果脚本已经下载完成过了，再遇到按需加载该脚本的地方，应当从缓存中取出结果，无需再下载
2. 如果脚本正在下载过程中，再遇到按需加载该脚本的地方, 应当监听该脚本的下载结果, 不应该再次发起脚本请求

我们今天来看看webpack是如何实现这样的逻辑的？

### 代码
#### 源代码
```
//index.js 
import('./a').then(data => {
  console.log(data)
})

//a.js
export default 'module a'
```

#### bundle.js
```
__webpack_require__
  .e("src_a_js") //chunk id
  .then(__webpack_require__.bind(__webpack_require__, "./src/a.js"))
  .then(function (data) {
    console.log(data);
  });
```


### 分析
我们可以看到webpack打包后的代码把import()语句换成了webpack自定义的webpack_require.e 函数，下面我们就从这个函数看起：
### webpack_require.e
```
__webpack_require__.f = {};

__webpack_require__.e = (chunkId) => {
  return Promise.all(Object.keys(__webpack_require__.f).reduce((promises, key) => {
    __webpack_require__.f[key](chunkId, promises);
    return promises;
  }, []));
};

var installedChunks = {
  "main": 0
};

__webpack_require__.f.j = (chunkId, promises) => {
    // JSONP chunk loading for javascript
    var installedChunkData = __webpack_require__.o(installedChunks, chunkId) ? installedChunks[chunkId] : undefined;
    if(installedChunkData !== 0) { // 0 means "already installed".

      // a Promise means "currently loading".
      if(installedChunkData) {
        promises.push(installedChunkData[2]);
      } else {
        if(true) { // all chunks have JS
          // setup Promise in chunk cache
          var promise = new Promise((resolve, reject) => (installedChunkData = installedChunks[chunkId] = [resolve, reject]));
          promises.push(installedChunkData[2] = promise);

          // start chunk loading
          var url = __webpack_require__.p + __webpack_require__.u(chunkId);
          // create error before stack unwound to get useful stacktrace later
          var error = new Error();
          var loadingEnded = (event) => {
            if(__webpack_require__.o(installedChunks, chunkId)) {
              installedChunkData = installedChunks[chunkId];
              if(installedChunkData !== 0) installedChunks[chunkId] = undefined;
              if(installedChunkData) {
                var errorType = event && (event.type === 'load' ? 'missing' : event.type);
                var realSrc = event && event.target && event.target.src;
                error.message = 'Loading chunk ' + chunkId + ' failed.\n(' + errorType + ': ' + realSrc + ')';
                error.name = 'ChunkLoadError';
                error.type = errorType;
                error.request = realSrc;
                installedChunkData[1](error);
              }
            }
          };
          __webpack_require__.l(url, loadingEnded, "chunk-" + chunkId, chunkId);
        } else installedChunks[chunkId] = 0;
      }
    }
};
```

在我们分析__webpack_require__.f.j这段逻辑之前，我们先把__webpack_require__.l这个函数弄清楚，代码如下所示：
```
/* webpack/runtime/load script */
(() => {
  /* 
    类型解释
    inProgress = {[url: string]: [function: event => void]}
  */
  var inProgress = {};  
  var dataWebpackPrefix = "webpack-study:";
  // loadScript function to load a script via script tag
  __webpack_require__.l = (url, done, key, chunkId) => {
          if(inProgress[url]) { inProgress[url].push(done); return; } //已有inprogress的了
          var script, needAttach;
          if(key !== undefined) {
            var scripts = document.getElementsByTagName("script");
            for(var i = 0; i < scripts.length; i++) {
                    var s = scripts[i];  //判读脚本是否已存在
                    if(s.getAttribute("src") == url || s.getAttribute("data-webpack") == dataWebpackPrefix + key) { script = s; break; }
            }
          }
          if(!script) {
            needAttach = true;
            script = document.createElement('script');

            script.charset = 'utf-8';
            script.timeout = 120;
            if (__webpack_require__.nc) {
                    script.setAttribute("nonce", __webpack_require__.nc);
            }
            script.setAttribute("data-webpack", dataWebpackPrefix + key);
            script.src = url;
          }
          inProgress[url] = [done];
          var onScriptComplete = (prev, event) => {
              // avoid mem leaks in IE.
              script.onerror = script.onload = null;
              clearTimeout(timeout);
              var doneFns = inProgress[url];
              delete inProgress[url];
              script.parentNode && script.parentNode.removeChild(script);
              doneFns && doneFns.forEach((fn) => (fn(event)));
              if(prev) return prev(event);
          };
          var timeout = setTimeout(onScriptComplete.bind(null, undefined, { type: 'timeout', target: script }), 120000);
          script.onerror = onScriptComplete.bind(null, script.onerror);
          script.onload = onScriptComplete.bind(null, script.onload);
          needAttach && document.head.appendChild(script);
  };
 })();
```

__webpack_require__.l这个函数是用来加载脚本的。

他首先定义了一个inProgress对象，请求的script的url作为key，script加载完成的回调函数作为value。这里我们需要注意，因为加载脚本是需要时间的，如果这中间发出了对同一个脚本发出了多个请求，那么这里就是一个数组。

然后又定义了一个onScriptComplete函数，我们用假代码来解释他的逻辑。
```
onScriptComplete{
  1：取消定时器。这个定时器是120s，如果脚本在120s之内没有完成加载，那么会触发timeout的错误
  2：取出对应的done函数数组，然后从inProgress对象里把对应的key删除掉
  3：把脚本从DOM里删除掉，因为脚本已经执行过了，所以是可以删除掉的, 也是让document文件更加干净
  4: 执行done的函数
  5：如果提供了script.onerror或者script.onload，执行函数
}
```
onScriptComplete主要做的就是执行脚本加载完成后的回调函数，并做了一些清理工作

接下来我们假代码来理一下__webpack_require__.l的逻辑
```
if(url已经存在于inProgress){
  把done函数push到对应的数组中
  返回
}else{
  let script
  if(脚本是否已经插入到document里面)
    script:= document里的对应script
  else
    script:= 通过DOM动态创建新的script
  inProgress[url] := done
  设置120s的定时器，若120s超时，则会抛出timeout的错误
  if(是动态创建的新脚本)
    document.head.appendChild(script);
}


了解了__webpack_require__.l，我们再看__webpack_require__.f.l。同样的我们也是用假代码来理一下逻辑：
```
/*
类型解释：
installedChunk = { [chunkId: string]: 0 | [resolve, reject, promise])}
如果chunkId对应的值是0，那么说明chunk已经加载完毕。没有加载完毕的话，对应存储的是一个promise
*/

installedChunkData := 根据chunkId从installedChunks取出对应的值
if(chunk还未加载完成){
  if(chunk的请求已经发出){
    把对应的promise放到数组中
  }else{ 
    //请求第一次发出
    构造加载的promise
    把promsie放入数组
    构造脚本对应的url
    定义加载完成的函数loadingEnded
    loadingEnded := function(){
      if(加载的promise还没有完成){
        promise reject, 抛出异常
      }
    }
    调用__webpack_require__.l函数
  }
}
```

我们再来看需要按需加载的chunk文件
```
(self["webpackChunkwebpack_study"] = self["webpackChunkwebpack_study"] || []).push([["src_a_js"],{ 
  "./src/a.js":
  ((__unused_webpack_module, __webpack_exports__, __webpack_require__) => {
    __webpack_require__.r(__webpack_exports__);
    __webpack_require__.d(__webpack_exports__, {
      "default": () => (__WEBPACK_DEFAULT_EXPORT__)
    });
    const __WEBPACK_DEFAULT_EXPORT__ = ('module a');
  })
}]);
```

```
var webpackJsonpCallback = (parentChunkLoadingFunction, data) => {
  var [chunkIds, moreModules, runtime] = data;
  // add "moreModules" to the modules object,
  // then flag all "chunkIds" as loaded and fire callback
  var moduleId, chunkId, i = 0;
  if(chunkIds.some((id) => (installedChunks[id] !== 0))) {
    for(moduleId in moreModules) {
      if(__webpack_require__.o(moreModules, moduleId)) {
        __webpack_require__.m[moduleId] = moreModules[moduleId];
      }
    }
    if(runtime) var result = runtime(__webpack_require__);
  }
  if(parentChunkLoadingFunction) parentChunkLoadingFunction(data);
  for(;i < chunkIds.length; i++) {
    chunkId = chunkIds[i];
    if(__webpack_require__.o(installedChunks, chunkId) && installedChunks[chunkId]) {
      installedChunks[chunkId][0]();
    }
    installedChunks[chunkId] = 0;
  }
}

var chunkLoadingGlobal = self["webpackChunkwebpack_study"] = self["webpackChunkwebpack_study"] || [];
chunkLoadingGlobal.forEach(webpackJsonpCallback.bind(null, 0));
chunkLoadingGlobal.push = webpackJsonpCallback.bind(null, chunkLoadingGlobal.push.bind(chunkLoadingGlobal));
```

这里的代码有点乱，但是实际上做的就一件事情，chunk文件加载完成后执行代码的时候，先resolve对应的promise，然后会把installedChunks里面对应的值为0。


研究完所有的代码，我们再来看一开始我们提出的问题：
1. 如果脚本已经下载完成过了，再遇到按需加载该脚本的地方，应当从缓存中取出结果，无需再下载
如果已经完成下载，那么installledChunks里面对应的值为0，__webpack_require__.f.l里的加载逻辑不会被执行

2. 如果脚本正在下载过程中，再遇到按需加载该脚本的地方, 应当监听该脚本的下载结果, 不应该再次发起脚本请求
如果脚本正在下载，那么在下载过程中同时发起的多个请求，都会使用installledChunks里面的同一个Promise，不会每次创建新的Promise。这个Promise会在加载这个chunk完成的时候被resolve






