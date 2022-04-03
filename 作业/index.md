# http强缓？本地缓存？workbox 项目中怎么利用 核心区别是什么。
http的强缓由三个属性共同决定Expires、Cache-Control、Pragma

Pragma(只能设置一个值，就是no-cache)

Cache-Control： max-age（缓存周期）、no-cache等

Expires 的值是一个 HTTP 日期，当服务器返回响应时，在 Response Headers 中将过期时间写入 Expires 字段。
在浏览器发起请求时，会根据系统时间和 Expires 的值进行比较，如果系统时间超过了 Expires 的值，缓存失效，会继续从服务器获取资源

Pragma(只能设置一个值，就是no-cache) > Cache-Control > Expires

# 本地缓存

cookie session localstorage

# workbox在项目中怎么用

原生的serverwork比较繁琐，需要自己去缓存内容，以及更新，在项目中一般使用workbox-webpack-plugin
```js
// 初始化完毕也需要在entry注册serviceWorker
const { GenerateSW } = require('workbox-webpack-plugin');
plugins:[new GenerateSW ({
  clientsClaim: true,
  skipWaiting: true
})]
```
 <!-- 原生 -->
```js
 if("serviceWorker" in navigator) {
    console.log("当前控制权",navigator.serviceWorker.controller);
    navigator.serviceWorker.register("/sw.js")
    .then(function(registration){
        console.log("serviceWorker注册成功",registration.scope);
    }).catch(function(err){
        console.log("serviceWorker注册失败",err);
    })
}

```
```js
// sw.js
var cacheName = "kk-v1";
//缓存的资源列表
var filesTocache = [
    "/scripts/index.js",
    "/images/timg.jpeg",
    "/scripts/test.js",
    ....
];
//安装阶段去定义怎么缓存当前的文件
self.addEventListener("install", function (event) {
    console.log("安装成功");
    event.waitUntil(updateStaticCache());
})

function updateStaticCache() {
    //对所有的静态资源进行缓存的过程
    return caches.open(cacheName)
        .then(function (cache) {
            return cache.addAll(filesTocache);
        }).then(() => self.skipWaiting());
}
//更新缓存的版本
self.addEventListener("activate", function (event) {
    event.waitUntil(caches.keys().then(function (keyList) {
        return Promise.all(keyList.map(function (key) {
            if (key !== cacheName) {
                return caches.delete(key);
            }
        }))
    }))
})
// 缓存中有从缓存中拿，没有的话就从fetch文件
self.addEventListener("fetch", function (event) {
    // event.respondWith(new Response("Hello World"))
    event.respondWith(
        caches.match(event.request).then(function(response){
            return response || fetch(event.request);
        })
    );
})

```
# 区别
强缓： 
一般用来缓存不轻易改变的资源，也就是说非业务资源，例如react、react-router-dom

本地缓存：
cookie一般用来鉴权，缓存一些用户标志等信息
localStorage可用来缓存常用的资源或者同源应用之间的业务标记
sessionStorage 当前网页会话下有效，关闭页面或浏览器后就会被清除，也是经常用来用来单页应用的业务资源存储或者业务标记

workbox：workbox 背后是 Service Worker 和 Cache API，解决的问题是站点提供离线访问能力。





