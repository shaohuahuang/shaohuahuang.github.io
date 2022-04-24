# yarn and yarn2.0

## Yarn

安装三个阶段：
- Dependency resolution  
确定所有依赖
- Fetch Packages  
把包下载到全局缓存里面
- Linking Packages  
从全局缓存拷贝到项目下面



### 安装版本统一

```
// 一个yarn.lock文件例子
copy-descriptor@^0.1.0:
  version "0.1.1"
  resolved "https://registry.yarnpkg.com/copy-descriptor/-/copy-descriptor-0.1.1.tgz#676f6eb3c39997c2ee1ac3a924fd6124748f578d"
  integrity sha1-Z29us8OZl8LuGsOpJP1hJHSPV40=

core-util-is@~1.0.0:
  version "1.0.2"
  resolved "https://registry.yarnpkg.com/core-util-is/-/core-util-is-1.0.2.tgz#b5fd54220aa2bc5ab57aab7140c940754503c1a7"
  integrity sha1-tf1UIgqivFq1eqtxQMlAdUUDwac=

create-ecdh@^4.0.0:
  version "4.0.3"
  resolved "https://registry.yarnpkg.com/create-ecdh/-/create-ecdh-4.0.3.tgz#c9111b6f33045c4697f144787f9254cdc77c45ff"
  integrity sha512-GbEHQPMOswGpKXM9kCWVrremUcBmjteUaQ01T9rkKCPDXfUHX0IoP9LpHYo2NPFampa4e+/pFDc3jQdxrxQLaw==
  dependencies:
    bn.js "^4.1.0"
    elliptic "^6.0.0"

```





## Reference
[https://hackernoon.com/working-of-yarn-and-npm-974b79f10341](https://hackernoon.com/working-of-yarn-and-npm-974b79f10341)

[node_modules困境](https://z.itpub.net/article/detail/E8D5F33F58F9FF55916877DB69F9573C)