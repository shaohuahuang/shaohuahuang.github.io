# 浏览器history API

## onhashchange
对于一个URL，如：http://github.com#abc123，“#”后面的abc123即为hash内容. 

Example:
```
window.onhashchange = function(event){
  console.log(event.oldURL, event.newURL);
  let hash = location.hash.slice(1)
}
```


## history
切换历史状态包括back, forward, go三种方法

修改历史状态，包括pushState, replaceState

onpopstate事件，切换的时候都会触发


## Reference
[深入理解 react-router 路由系统](https://zhuanlan.zhihu.com/p/20381597)