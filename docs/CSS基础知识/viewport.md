# viewport

在 PC 端，视口指的是浏览器的可视区域，其宽度和**浏览器窗口的宽度**保持一致。

而移动端则较为复杂，它涉及到三个视口：  
布局视口（Layout Viewport）  
视觉视口（Visual Viewport）  
理想视口（Ideal Viewport）  


## 基本概念

### 物理像素 (设备像素，device pixels)
指的是设备屏幕的物理像素，任何设备的物理像素数量都是固定的。

### CSS像素
CSS文件里定义的1px并不一定对应1px的物理像素，他是由屏幕的设备像素比例(device pixel ratio)以及用户的缩放来决定的
举个例子：
假如屏幕的设备像素比例为2，用户放大了屏幕2倍，那么CSS像素就是物理像素的4倍。

### 屏幕大小
屏幕的空间大小，用英寸表示

### 屏幕分辨率
屏幕的宽有多少像素，屏幕的高有多少像素，320 * 480

### DPI/PPI （dots per inch)
一英寸的屏幕，里面包含了多少物理像素

### Point
苹果手机用Point来兼容所有不同屏幕分辨率的机型。Point的空间大小是固定的，不随其他而变化
换算公式：1px = 1Point * DPI/163

### DIP (Density Independent Pixel)
类似于Point，DIP是安卓手机里单位，它的空间大小也是固定的。
换算公式：1px = 1DIP * DPI/160

### vh/vw
这个单位是相对于布局视口来计算的

## 移动端视口

### 布局视口
一般移动设备的浏览器都默认设置了一个 viewport 元标签，定义一个布局视口（layout viewport）  
布局视口的宽度/高度可以通过 document.documentElement.clientWidth / Height 获取。

我们也可以设置布局视口的宽度
```
<meta name="viewport" content="width=400">
// 这样会把视口宽度改为400px
```
> viewport元标签只对移动端浏览器生效

### 理想视口
这个视口的大小是由设备商决定的。比如iphone4s，不管是否有Retina，理想视口都是320*480。因为这对页面来说是最理想的

```
<meta name="viewport" content="width=device-width">
// 使布局视口与理想视口的宽度一致
```

### 视觉视口
视觉视口是用户当前看到的区域，用户可以通过缩放操作视觉视口，同时不会影响布局视口。
视觉视口和缩放比例的关系为：
> 当前缩放值 = 理想视口宽度  / 视觉视口宽度


Note: 
> CSS像素是与Ideal Viewport的单位保持一致的。也就是说如果把Ideal Viewport设成400px，那么CSS像素设成400px，这个元素就会占据整个屏幕宽度