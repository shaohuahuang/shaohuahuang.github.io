## axios拦截器和请求取消 

### 拦截器
axios提供了拦截器功能，可以在request或response被then和catch处理之前，进行一些逻辑处理

```javascript 
// 设置请求拦截器
axios.interceptors.request.use(
  config => {
    // console.log(config) // 该处可以将config打印出来看一下，该部分将发送给后端（server端）
    config.headers.Authorization = store.state.token
    return config // 对config处理完后返回，下一步将向后端发送请求
  },
  error => { // 当发生错误时，执行该部分代码
    // console.log(error) // 调试用
    return Promise.reject(error)
  }
)

// 定义响应拦截器 -->token值无效时,清空token,并强制跳转登录页
axios.interceptors.response.use(function (response) {
  // 响应状态码为 2xx 时触发成功的回调，形参中的 response 是“成功的结果”
  return response
}, function (error) {
  // console.log(error)
  // 响应状态码不是 2xx 时触发失败的回调，形参中的 error 是“失败的结果”
  if (error.response.status === 401) {
    // 无效的 token
    // 把 Vuex 中的 token 重置为空，并跳转到登录页面
    // 1.清空token
    store.commit('updateToken', '')
    // 2.跳转登录页
    router.push('/login')
  }
  return Promise.reject(error)
})
```

如上面代码所示，在请求发出之前，我们可以获取config，修改config的headers等其他信息。在response返回后，我们可以对response进行预处理。一些通用的逻辑可以在这里处理，避免逻辑的重复。 


### 取消请求
在有些场景下，我们需要取消正在发出的请求，避免一些奇怪行为的出现。这些场景包括：
- 用户想主动取消请求
- 用户离开一个页面的时候取消请求
- 当输入框内容变化时，取消正在发出的请求，然后发一个新的请求

在axios@0.22.0之前的版本中，使用的是CancelToken来取消，而在之后的版本会使用fetch API已经支持的Abortcontroller来实现。

我们来考虑这样一个场景，假设有一个搜索框，用户改变搜索内容时，需要先把之前的搜索请求取消掉，然后发出新的请求。在这个场景下，我们看看两种不同的实现方法。

#### CancelToken
axios可以利用CancelToken创建出一个source, 发出请求的时候可以在请求的option里添加source.token, 将请求与该source绑定。之后要取消请求的话，可以直接调用source.cancel方法。

我们来看一下简单的代码实现

```javascript
let source;

function searchItems(event) {
  const name = event.target.value;
  
  // cancel the request
  if (source) source.cancel('Operation canceled by the user.');
  
  source = axios.CancelToken.source();

  const option = {
    params: { name }, // Pass query strings via param option
    cancelToken: source.token // add the cancel token, as an option for the request
  };
  
  axios.get('https://example.com', option)
    .then((response) => {
      console.log(response)
    })
    .catch(function (thrown) {
    if (axios.isCancel(thrown)) {
      console.log('Request canceled', thrown.message);
    } else {
        // handle errors
    }
 });
}
```

#### Abortcontroller

Abortcontroller的实现与上面大致相似 
首先先创建一个controller实例，然后在请求的option里添加signal将controller与请求绑定，最后可以通过controller.abort()来取消请求
```javascript
let controller;

function searchItems(event) {
  const name = event.target.value;
  
  // cancel the request
  if (source) controller.abort('Operation canceled by the user.');
  
  controller = new AbortController();

  const option = {
    params: { name }, // Pass query strings via param option
    signal: controller.signal // add the cancel token, as an option for the request
  };
  
  axios.get('https://example.com', option)
    .then((response) => {
      console.log(response)
    })
    .catch(function (thrown) {
      // handle errors
 });
}
```


## Reference
[axios 请求拦截器&响应拦截器](https://juejin.cn/post/7100470316857557006)

[Abortcontroller and Axios cancel token](https://03balogun.medium.com/practical-use-case-of-the-abortcontroller-and-axios-cancel-token-7c75bf85f3ea)