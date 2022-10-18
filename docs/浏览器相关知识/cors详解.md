## CORS详解
> CORS（Cross-origin resource sharing） “跨域资源共享”

### JSONP 
全称： JSON with padding
在cors出现之前，我们都是用JSONP发送跨域请求。

JSONP可以跨域请求的原因是：script标签没有同源策略的限制，网页可以得到从其他来源动态产生JSON数据。
用 JSONP 抓到的并不是 JSON，而是任意的JavaScript，用 JavaScript 直译器执行而不是用 JSON 解析器解析。

举个例子：
```
<script type="text/javascript" src="http://169.254.200.238:8080/jsonp.do" />
```
这个script可以跨域的发出请求，获取对应的数据。

这里还有一个问题需要解决，如果直接执行JSON数据，会报错。所以服务端其实不是直接返回数据，而是返回callback函数的调用。这个callback是前后端约定好的，前端需要定义好这callback, 这样JSONP的response返回后，可以直接执行。

### CORS
JSONP不是一个优雅的解决跨域的方法，后来CORS这个W3C标准应运而生。

整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，**CORS通信与同源的AJAX通信没有差别**，代码完全一样

#### 两种请求
浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

只要同时满足以下两大条件，就属于简单请求，其他则是非简单请求。
```
(1) 请求方法是以下三种方法之一：
HEAD
GET
POST

(2）HTTP的头信息不超出以下几种字段：
Accept
Accept-Language
Content-Language
Last-Event-ID
Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
```

#### 简单请求
对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。
服务器根据这个值，决定是否同意这次请求。  

如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段，就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。
> 注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。
```
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
```

这里有三个与CORS相关的字段： 
- Access-Control-Allow-Origin  
  该字段是**必须的**。它的值要么是请求时Origin字段的值，要么是一个*，表示接受任意域名的请求。
- Access-Control-Allow-Credentials  
  该字段**可选**。它的值是一个布尔值，表示是否**允许发送Cookie**。默认情况下，**Cookie不包括在CORS请求之中**。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。
  > 这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。
- Access-Control-Expose-Headers   
  该字段**可选**。CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。  
  如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。上面的例子指定，getResponseHeader('FooBar')可以返回FooBar字段的值。

##### withCredentials 属性
CORS请求默认不发送Cookie和HTTP认证信息。如果要把Cookie发到服务器，一方面要服务器同意，指定Access-Control-Allow-Credentials字段。
另一方面，开发者必须在AJAX请求中打开withCredentials属性。 
> 需要注意的是，如果要发送Cookie，Access-Control-Allow-Origin就**不能设为星号**，必须指定明确的

#### 非简单请求
非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些**HTTP动词**和**头信息字段**。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

举个例子：
```
// 跨域请求代码
var url = 'http://api.alice.com/cors';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();


// preflight http请求的头信息
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...

// preflight http response 
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

preflight请求里面有两个特殊字段：
- Access-Control-Request-Method
该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是PUT。

- Access-Control-Request-Headers
该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段，上例是X-Custom-Header。

preflight响应里的特殊字段：

- Access-Control-Allow-Methods 
该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。

- Access-Control-Allow-Headers  
如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。

- Access-Control-Allow-Credentials  
该字段与简单请求时的含义相同。

- Access-Control-Max-Age  
该字段**可选**，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求。



## Reference
- https://juejin.cn/post/6844903476594475016 
- [JSONP详解](https://zhuanlan.zhihu.com/p/24390509)
- [阮一峰 - CORS](https://www.ruanyifeng.com/blog/2016/04/cors.html)