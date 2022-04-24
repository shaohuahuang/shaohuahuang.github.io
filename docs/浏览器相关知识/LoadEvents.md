# 浏览器里的Load Events

## DOMContentLoaded
触发时机：当HTML被完全加载和解析，但是不必等(css, images, iframe)完成加载 

Note: css下载会阻塞js的执行，如果css脚本在js之前，那么就需要等这个css下载完毕，js执行完毕，事件才会触发。

## load
触发时机：当页面被完全加载，所有依赖的资源包括(css, images等)