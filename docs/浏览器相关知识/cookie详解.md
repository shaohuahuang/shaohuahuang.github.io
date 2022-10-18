## Cookie详解

### Cookie里的属性 

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


## Reference 
https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies