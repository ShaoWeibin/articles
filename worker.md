# <center>Web Workers & Service Workers 入门实践</center>

## 写在前面
为了提高 app 用户体验，提升数据加载效率，目前 web 编辑器项目中已经使用 Serveice Workers 进行任务数据的离线缓存，后续其他项目会逐步引入 Service Workers，因此有必要做个入门总结。
如果你想知道如何使用 Web Workers 和 Service Workers，此文会对你有所帮助。

## Web Workers

### 概念

web worker 是运行在后台的 JavaScript，独立于其他脚本，不会影响页面的性能。

---

### 分类
- *Dedicated Workers*  
> 专用的 worker，只能被创建它的 JS 访问，创建它的页面关闭，它的生命周期就结束了。
- *Shared Workers*  
> 共享的 worker，可以被同一域名下的 JS 访问，关联的页面都关闭时，它的生命周期就结束了。

---

### 浏览器兼容性
![support](https://github.com/ShaoWeibin/images/blob/master/worker-support.png?raw=true)

---

### 实践
#### worker 特性检测
``` javascript
if (window.Worker) {
  ...
}
```

#### 创建 worker
``` javascript
const worker = new Worker('/worker.js');
```

#### 事件处理函数
- *main script*
``` javascript   
worker.onmessage = function(msg) {
  console.log('Message received from worker script');
  console.log(msg.data);
  worker.postMessage('I am main script');
}

/**
 * @param message 可读性良好的错误消息
 * @param filename 发生错误的脚本文件名
 * @param lineno 发生错误时所在脚本文件的行号
 */
worker.onerror = function(error) {
  console.log(error.filename, error.lineno, error.message);
  throw error;
}
```

- *worker*
``` javascript 
onmessage = function(msg) {
  console.log('Message received from main script');
  console.log(msg.data);
  postMessage('I am worker script');
}

onerror = function(error) {
  console.log(error);
  throw error;
}
```

#### 终止 worker
- *main script*
``` javascript
worker.terminate();
```

- *worker*
``` javascript
self.close();
```

#### 引入脚本和库
Worker 线程能够访问一个全局函数 importScripts() 来引入脚本，该函数接受 0 个或者多个 URI 作为参数来引入资源。
``` javascript
importScripts();                        /* 什么都不引入 */
importScripts('foo.js');                /* 只引入 "foo.js" */
importScripts('foo.js', 'bar.js');      /* 引入两个脚本 */
```
> 注意： 脚本的下载顺序不固定，但执行时会按照传入 importScripts() 中的文件名顺序进行。这个过程是同步完成的；直到所有脚本都下载并运行完毕， importScripts() 才会返回。

#### 生成 subworker
如果需要的话 worker 能够生成更多的 worker。这就是所谓的 subworker，它们必须托管在同源的父页面内。而且，subworker 解析 URI 时会相对于父 worker 的地址而不是自身页面的地址。这使得 worker 更容易记录它们之间的依赖关系。

#### worker 中数据的接受和发送
- *在主页面 与 worker 之间传递的数据是通过拷贝，而不是共享来完成的*
>传递给 worker 的对象需要经过序列化，接下来在另一端还需要反序列化。页面与 worker 不会共享同一个实例，最终的结果就是在每次通信结束时生成了数据的一个副本。大部分浏览器使用结构化拷贝来实现该特性。  
结构化克隆算法本质是将原始对象的所有字段的值复制到新对象里。如果一个字段是对象，这些字段会被递归复制，直到所有的字段和子字段都被复制进新的对象里。

- *通过转让所有权(可转让对象)来传递数据*
> Google Chrome 17 与 Firefox 18 包含另一种性能更高的方法来将特定类型的对象(可转让对象) 传递给一个 worker/从 worker 传回 。可转让对象从一个上下文转移到另一个上下文而不会经过任何拷贝操作。这意味着当传递大数据时会获得极大的性能提升。一旦对象转让，那么它在原来上下文的那个版本将不复存在。该对象的所有权被转让到新的上下文内。

``` javascript
// Create a 32MB "file" and fill it.
var uInt8Array = new Uint8Array(1024*1024*32); // 32MB
for (var i = 0; i < uInt8Array .length; ++i) {
  uInt8Array[i] = i;
}

worker.postMessage(uInt8Array.buffer, [uInt8Array.buffer]);
```

---

### 局限性
- *同源限制*
> worker 不能跨域加载 js。
- *可操作对象限制*
> * 可操作对象: Navigator, location, XMLHttpRequest, setTimeout/setInterval, Application Cache 等。
> * 不可操作对象: DOM, window, document, parent。
- *文件限制*
> 子线程无法读取本地文件，即 worker 只能加载网络文件。
- *兼容性问题*
> 不是每个浏览器都支持这个新特性，且各个浏览器对 worker 的实现不大一致，有些浏览器里允许 worker 中创建新的 worker,有些浏览器就不允许（如 Chrome）。

---

### 使用场景
- *数学运算*
- *大量数据检索*
- *图像处理*
- *背景数据处理*

---

### 总结

---

## Service Workers

### 背景  

#### 如何将网页的用户体验做到极致
- *应用秒开*
- *离线也能浏览*
- *应用可以增量更新*

#### 目前 web 应用缓存的解决方案与不足
- *基于浏览器头信息的缓存方式*
> http头部的Etag和Last-Modified是否修改，修改则重新请求，否则忽略；当然还有一种根据expires过期时间来判断的，原理一样，但是这两种方法都必不可少的至少会产生大量http请求，即使返回304。而且一旦离线，浏览器就无计可施了。
- *使用 APP Cache 缓存*
> h5 提供了 App cache 来解决静态文件存储的问题，它通过将要缓存的静态文件声明在一个 manifest 文件清单里，然后在要缓存的 html 里通过 manifest 属性关联清单文件即可在下次载入 html 时优先加载缓存清单中列出的静态文件。
> 对manifest文件更新，会重新请求所有文件，实际上可能只更新了很少量文件。
- *使用 localstorage 存储*
> 一是无法对静态文件进行存储，二是大小限制。
- *indexedDB 缓存*
> 容量足够，存取自由，可以离线缓存，对静态资料的缓存极其麻烦。

--- 

### 概念  

Service worker 是一个注册在指定源和路径下的事件驱动 worker。它采用 JavaScript 控制关联的页面或者网站，拦截并修改访问和资源请求，细粒度地缓存资源。你可以完全控制应用在特定情形（最常见的情形是网络不可用）下的表现。

--- 

### 特点
- *https 环境 / localhost / 127.0.0.1*
- *运行于浏览器后台(独立进程)，可以控制打开的作用域范围下所有的页面请求*
- *单独的作用域范围，单独的运行环境和执行线程*
- *能向客户端推送消息*
- *不能操作页面 DOM。但可以通过事件机制间接操作*
- *一旦被 install，就永远存在，除非被手动 unregister*
- *异步实现，内部大都是通过 Promise 实现*

--- 

### 浏览器兼容性
![support](https://github.com/ShaoWeibin/images/blob/master/sw-support.png?raw=true)

--- 

### 生命周期
![image](https://mdn.mozillademos.org/files/12630/important-notes.png)
![image](https://mdn.mozillademos.org/files/12636/sw-lifecycle.png)

--- 

### 实践
#### worker 特性检测
``` javascript
if (window.serviceWorker) {
  ...
}
```

#### register
``` javascript
/**
 * 注册 serviceWorker
 */
export function registerServiceWorker() {
  if (navigator.serviceWorker) {
    // 第二个参数为 scope
    navigator.serviceWorker.register('/serviceWorker.js', {scope: '/'})
    .then(registration => {
      console.log('Service Worker register success! Scope is' + registration.scope);
    })
    .then(() => {
      
    })
    .catch(error => {
      console.log(error);
    });
  }
  else {
    console.log('Sorry, your browser does not support Service Workers...');
  }
}
```

#### unregister
``` javascript
/**
 * 注销 serviceWorker
 */
export function unregisterServiceWorker() {
  if (navigator.serviceWorker) {
    navigator.serviceWorker.ready.then(registration => {
      registration.unregister();
    });
  }
}
```

#### install
``` javascript
/**
 * install
 */

const CACHE_NAME = 'sw-v1';
const urlsToCache = [
  '/',
  'static/js/bundle.js',
  '/worker.js',
  '/serviceWorker.js'
];

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
    .then(cache => {
      return cache.addAll(urlsToCache);
    })
  );
});
```

#### activate
``` javascript
/**
 * activate
 */
self.addEventListener('activate', event => {

});
```

#### fetch
``` javascript
/**
 * fetch
 */
self.addEventListener('fetch', event => {
  const { request } = event;
  
  event.respondWith(
    caches.match(request).then(response => {
      // return matched.
      if (response) {
        return response;
      }

      // fetch and save it into cache.
      fetch(request).then(response => {
        return caches.open(CACHE_NAME).then(cache => {
          cache.put(request, response.clone());
          return response;
        });
      });
    }).catch(e => {
      console.log('fetch error');
      return e;
    })
  );
});
```

#### message
- *main script message*
``` javascript
/**
 * 页面发送给 serviceWorker
 * @param {*} msg 
 */
export function postMessage(msg) {
  // 获取 ServiceWorker 句柄
  // 只有注册成功后该句柄才会存在
  const controller = navigator.serviceWorker.controller;
  if (controller) {
    controller.postMessage(msg, []);
  }
}

/**
 * 页面从 serviceWorker 接受消息
 */
function onMessage() {
  if (navigator.serviceWorker) {
    navigator.serviceWorker.addEventListener('message', event => {
      console.log('Recieve message from serviceWorker');
      console.log(event.data);
    });
  }
}
```

- *serviceWorker message*
``` javascript
/**
 * message
 * 监听 message 事件获取页面发送的消息
 */
self.addEventListener('message', event => {
  console.log(event.data);
});

/**
 * serviceWorker 发送消息给页面
 * 页面句柄从 self.clients 中得到
 */
self.clients.matchAll().then(clientList => {
  clientList.forEach(client => {
    client.postMessage('Hi, I am send from Service worker！')
  });
});
```

#### push
``` javascript

```
#### sync
``` javascript

```

#### online / offline
当网络状态发生变化时，会触发 online 或 offline 事件。
``` javascript
self.addEventListener('online', () => {
  console.log('online');
});

self.addEventListener('offline', () => {
  console.log('offline');
});
```

#### unhandledrejection
``` javascript
/**
 * unhandledrejection
 * Promise 类型的回调发生 reject 却没有 catch 处理, 会触发该事件
 */
self.addEventListener('unhandledrejection', event => {
  console.log(event);
});

```

#### onerror
``` javascript
/**
 * error 事件
 * JS 执行发生错误, 会触发 error 事件
 * @param {*} errorMessage 
 * @param {*} scriptURI 
 * @param {*} lineNumber 
 * @param {*} columnNumber 
 * @param {*} error 
 */
self.onerror = function(
  errorMessage,
  scriptURI, 
  lineNumber, 
  columnNumber, 
  error
) {
  if (error) {
    console.log(error);
  }
  else {
    console.log(errorMessage, scriptURI, lineNumber, columnNumber);
  }
}
```

#### 版本更新
如果你的 service worker 已经被安装，但是刷新页面时有一个新版本的可用，新版的 service worker 会在后台安装，但是还没激活。当不再有任何已加载的页面在使用旧版的 service worker 的时候，新版本才会激活。一旦再也没有更多的这样已加载的页面，新的 service worker 就会被激活。
![debug](https://github.com/ShaoWeibin/images/blob/master/

##### 自动更新
如果希望在有了新版本时，所有的页面都得到及时自动更新怎么办呢？可以在 install 事件中执行 self.skipWaiting() 方法跳过 waiting 状态，然后会直接进入 activate 阶段。接着在 activate 事件发生时，通过执行 self.clients.claim() 方法，更新所有客户端上的 Service Worker。

``` javascript
self.addEventListener('install', event => {
  event.waitUntil(self.skipWaiting());
  console.log('Service Worker install success!');
});

self.addEventListener('activate', event => {
  const cacheWhitelist = [CACHE_NAME];

  event.waitUntil(
    Promise.all([
      // 更新所有客户端
      self.clients.claim(),
      // 清理旧版本缓存
      caches.keys().then(function(keyList) {
        return Promise.all(keyList.map(function(key) {
          if (cacheWhitelist.indexOf(key) === -1) {
            return caches.delete(key);
          }
        }));
      })
    ])
  );
  console.log('Service Worker activate success!');
});
```

##### 手动更新

``` javascript
const version = '1.0.1';

navigator.serviceWorker.register('/serviceWorker.js', {scope: '/'})
.then(registration => {
  if (localStorage.getItem('sw_version') !== version) {
    registration.update().then(() => {
      localStorage.setItem('sw_version', version);
    });
  }
});
```

##### 强制更新
之后至少每24小时它会被下载一次。它可能被更频繁地下载，不过每24小时一定会被下载一次，以避免不良脚本长时间生效。

--- 

### Chrome 调试
> - 使用 Chrome 浏览器，可以通过 Application -> Service Workers 面板查看和调试。
![debug](https://github.com/ShaoWeibin/images/blob/master/
> -在 tab 页通过 chrome://serviceworker-internals/ 可查看所有的 Service Workers。
![debug](https://github.com/ShaoWeibin/images/blob/master/
--- 

### 使用场景
- *离线缓存*
- *消息推送*
- *静默更新*

---

### 总结

---

## 总结

Web Workers 旨在解决耗时的 js 执行影响 ui 响应的问题。  
Service Workers 是为了解决 Web App 的用户体验不如 Native App 的问题而提供的一系列技术集合。

--- 

## 参考文章
> http://imweb.io/topic/56592b8a823633e31839fc01  
https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers  
https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API  
https://juejin.im/post/599163316fb9a03c3c14d751  
http://lzw.me/a/pwa-service-worker.html  
https://zhuanlan.zhihu.com/p/27264234  

---
