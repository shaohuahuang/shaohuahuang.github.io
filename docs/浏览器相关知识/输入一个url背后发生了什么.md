# 输入一个url背后发生了什么

1. 对url进行DNS解析，获得IP
2. 向IP对应的服务器请求html文件
3. 开始解析html文件，构建DOM树
4. 遇到CSS文件，则下载该文件，此时会阻塞html的解析。下载完毕执行CSS, 此时不会阻塞html的解析
5. 遇到JS文件，下载并立即执行JS文件，下载和执行都会阻塞html的解析。因此为了减少白屏时间，通常我们会把JS文件放在最底部
6. 根据HTML我们构建了DOM树，根据CSS我们构建了CSSDOM树，两者合并就行成我们的渲染树(render tree)
7. 进行布局，根据文档流，盒模型，计算出大小和位置
8. Paint绘制，把边框，颜色，文字颜色，阴影等画出来
9. Composite合成，根据层叠关系展示画面

## 浏览器构成
- 浏览器主进程：  
  负责浏览器界面显示，用户交互，子进程管理

- GPU进程  
  3D的渲染

- 网络进程  
  负责页面的网络资源下载

- 渲染进程（每个tab都有一个）  
  核心任务是将 HTML、CSS 和 JavaScript 转换为用户可以与之交互的网页，排版引擎 Blink 和 JavaScript 引擎 V8 都是运行在该进程中
  (Firefox use Gecko, Safari use Webkit, Chrome use Blink - fork of webkit)

## js和css脚本处理的顺序
### js脚本
当HTML parser解析到script的时候，会立即下载脚本并且执行。这个会阻塞HTML的解析。  
当然我们可以通过添加defer和async参数来避免这种情况。  
defer：在HTML解析完毕之后再执行脚本  
async: 新开一个线程，来执行脚本  

### css脚本
CSS脚本不会改变DOM树结构，所以似乎不需要阻塞HTML的解析。
但是如果有JS脚本需要获取style的时候，这时就需要等CSS先解析好，才不会出错。
## Reference
[浏览器是如何工作的](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)