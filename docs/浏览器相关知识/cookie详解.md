## Cookie详解

### Cookie里的属性 
#### HttpOnly
cookie在JS不能被document.cookie获取和修改，增强安全性

#### Secure
标记为Secure的cookie只有通过https协议才能发送给服务端。

#### Domain属性
Domain指定了哪些主机可以接受Cookie，如果不指定，默认为origin，**不包含子域名**。如果指定了domain, 则一般包含子域名。因此，制定Domain比省略它的**限制要少**。当子域需要共享有关用户的信息时，这可能会有所帮助。

例如，如果设置 Domain=mozilla.org，则 Cookie 也包含在子域名中（如developer.mozilla.org）。

#### Path属性
Path 标识指定了主机下的哪些路径可以接受 Cookie，**子路径也会被匹配**

例如，设置 Path=/docs，则以下地址都会匹配：

/docs
/docs/Web/
/docs/Web/HTTP

#### SameSite属性
SameSite Cookie 允许服务器要求某个 cookie 在**跨站请求时不会被发送**

SameSite 可以有下面三种值：
- None。浏览器会在同站请求、跨站请求下继续发送 cookies，不区分大小写。
- Strict。浏览器将只在访问相同站点时发送 cookie。（在原有 Cookies 的限制条件上的加强，如上文“Cookie 的作用域”所述）
- Lax。与 Strict 类似，但用户从外部站点导航至 URL 时（例如通过链接）除外。在新版本浏览器中，为默认选项，Same-site cookies 将会为一些跨站子请求保留，如图片加载或者 frames 的调用，但只有当用户从外部站点导航到 URL 时才会发送。如 link 链接
> sameSite在chrome v80中提出，v86中全量默认设置sameSite为**Lax**

##### SameSite和同源的区别
同源策略是比较严苛的，需要协议 + 域名 + 端口完全一致，才认为是同源。  
SameSite相对于来说稍微宽泛一点。

在判断什么是SameSite，我们先来看一下Site的定义。  
**Site 定义**： 公共后缀和它前面那个名称的结合 [出处](https://link.juejin.cn/?target=https%3A%2F%2Fweb.dev%2Fsamesite-cookies-explained%2F%23explicitly-state-cookie-usage-with-the-samesite-attribute)  
**公共后缀(Public Suffix List)**： 互联网维护了所有的公共后缀列表[链接](https://link.juejin.cn/?target=https%3A%2F%2Fpublicsuffix.org%2Flist%2Fpublic_suffix_list.dat)

这里我们还要解释一下公共后缀和顶级域名的区别。我们了解的.com, .io都是顶级域名，而像.github.io是公共后缀。由此可见顶级域名与公共后缀并没有什么联系。

我们通过下面几个实际例子来帮助我们理解SameSite，假设下面的都是**用同样的协议**。如果协议不同，那么一定不是SameSite
```
1. a.taobao.com 和 b.taobao.com 是 SameSite。
   因为这里只有.com是公共后缀，所以根据定义，他们其实都属于站点.taobao.com

2. a.github.io 和 b.github.io 不是SameSite
   因为这里.github.io是公共后缀，所以这两个域名对应不同的站点

```

##### withCredentials和SameSite设置的值出现冲突的情况
我们再[cors详解](浏览器相关知识/cors详解.md)一文里面讲过，跨域的情况下，请求默认是不携带cookie的。那么如果我们把withCredentials设为true，允许携带cookie。但是我们cookie的SameSite设为strict，这个时候cookie会被发出去吗？ 
答案是不会的，SameSite拥有更高的优先级。 
譬如在a.demo2.com域名下请求a.demo.com的API，这个时候如果SameSite为Strict或者Lax，**那么cookie是不能被携带的** 

#### Expires属性
定义了cookie会在什么时候过期。

#### Max-Age
定义了cookie有效的时间


## Reference 
https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies