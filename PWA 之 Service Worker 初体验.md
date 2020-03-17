### 引言

本文是在探索 Service Worker 时总结的文章，首先会给大家展示 Service Worker 到底可以做什么，其次会详细展示它的配置文件和生命周期，大家如果有兴趣，可以访问：[https://github.com/banzhuanmei/test-sw](https://github.com/banzhuanmei/test-sw) 查看源代码。

**注：Service Worker 只能在`https`和`localhost`下运行！** 大家可以利用 github 提供的 `https` 环境查看效果。

**阅读引导：** 为避免文章冗余，代码部分我尽量在注释处说明每一步的功能，需要特殊说明的会单独介绍，请知悉！

# Service Worker 离线缓存
[https://banzhuanmei.github.io/test-sw/](https://banzhuanmei.github.io/test-sw/)，大家可以打开这个地址，试着把网断掉，或模拟手机 `offline` 模式重新刷新页面，会看到断网情况下，页面依然可以访问，是不是很开心～

这就是基于 Service Worker 实现的离线缓存效果，接下来看一下它的实现过程。

**注：** 实现过程包括注册 Service Worker、配置缓存、更新缓存、应用缓存，这其中所有 api 返回的都是`promise`对象，大家看的时候需要习惯再习惯，因为我一开始非常不适应，有一些障碍。

## 给应用配置缓存

这个 demo 比较简单，只需要缓存页面的样式和用到的图片，配置如下图：

```
// 缓存版本
const CACHE_NAME = "v5";
// 需要缓存的资源列表
const urlsToCache = [
  '/test-sw/',
  '/test-sw/css/style.css',
  '/test-sw/images/banner.png',
  '/test-sw/data.js',
  '/test-sw/images/waiting.png'
];
// self 为当前 scope 内的上下文
// 此处 event 是 ExtendableEvent 接口，继承了 Event。
self.addEventListener('install', event => {
// ExtendableEvent.waitUntil 延长了 install 事件过程，扩展了事件的生命周期，直到 caches.open() 传递的 Promise 被成功 resolve，确保所有要被缓存的文件都添加进来才去 install。
  event.waitUntil(
	// 打开一个缓存空间
	caches.open(CACHE_NAME).then(cache => {
	  console.log(cache);
	  // 添加要缓存的资源列表
	  return cache.addAll(urlsToCache);
	})
  );
});

```

这段代码中，大家可能不熟悉的有`caches`的方法 和 `self` 在此处的使用，下面我用两段内容分别说明一下。

`caches` 是在 Service Worker 的规范中定义的，它可以保存每个 Service Worker 申明的 cache 对象，有`open`、`match`、`has`、`delete`、`keys`五个核心方法，**注：虽然`caches`被定义在 Service Worker 的标准中，但它暴露在 window 中，可以单独使用，提供存储机制**，详细内容大家有兴趣的话可以学习下[它的 API](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache)，我这边只做简单介绍，保证大家顺利读下去。

此处的`self`，是用来访问全局上下文的，可能大家对这种使用会有疑问：全局为什么不使用 `window`？主要还是我没见过用 `self` 访问全局……因为 Service Worker 工作的线程并不在浏览器窗体线程中，它有自己独立的线程，所以无法访问`window`，无法访问`DOM`。

## 注册 Service Worker
如何使刚才配置的缓存生效呢？需要注册一个 Service Worker，代码如下：

```
if(navigator && navigator.serviceWorker) { // 检测浏览器是否支持 serviceWorker
  window.addEventListener('load', function() {
  // 注册一个 Service Worker 对象
    navigator.serviceWorker.register('/test-sw/service-worker.js')
    .then(function(registration) {
      console.log('注册成功');
      console.log(registration);
    }).catch(function(err) {
      console.log(err);
    });
  });
}
```

注册完后，可以在 Application 中的 Service Workers 中看到注册成功的信息：
![](https://banzhuanmei.github.io/test-sw/images/register.png)

此处我们单独介绍`navigator.serviceWorker`，`navigator.serviceWorker`返回一个 `ServiceWorkerContainer`对象，它提供了对 Service Worker 的注册、卸载、更新和访问。

`ServiceWorkerContainer.register(url,scope)`方法注册了一个 Service Worker，

* `url `代表`service-worker.js`的文件地址
* `scope `是`service-worker.js`的作用域，即缓存可以被应用的地方，省略时，作用域为`service-worker.js`所在的目录，比如上面代码中的`/test-sw/service-worker.js`，作用域为`/test-sw/`。
* 它返回一个 `promise` 对象，成功时会得到`registration`对象，打印出来，可以看到它内部如下：

![](https://banzhuanmei.github.io/test-sw/images/ServiceWorkerRegistration.png)


# Service Worker 生命周期
第一部分展示了 Service Worker 注册、安装、配置，这部分给大家详细展示 Service Worker 的生命周期和对缓存的管理。这里做了一张图，附上生命周期的关键点和 `actived` 时管理缓存的代码处理。

![](https://banzhuanmei.github.io/test-sw/images/service-worker-life.png)

## actived，管理缓存
一般在这个阶段执行管理 cache 的操作，例如清除掉旧缓存：

```
// 缓存更新
self.addEventListener('activate', function(event) {
  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.map(function(cacheName) {
          // 如果当前版本和缓存版本不一致
          if (cacheName !== CACHE_NAME) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

## 注销 Service Worker
### 手动注销
在浏览器中手动点 Unregister 按钮可手动删除已注册的 Service Worker，再次刷新时会看到激活时间已更新，并且修改 service-worker.js 之后，没有激活时，多次刷新显示时间不变，一直使用的是正激活的 Service Worker 版本，如图：

![](https://banzhuanmei.github.io/test-sw/images/unregister.png)

### 代码注销实现

为了更好的理解 Service Worker 的生命周期，这部分代码我是最后添加上去的，不过后来发现其实一开始看到这个效果更有助于理解：

```
self.addEventListener('install', event => {
  event.waitUntil(
	caches.open(CACHE_NAME).then(cache => {
	  console.log(cache);
	  return cache.addAll(urlsToCache);
	}).then(() => { 
	// 在最早注册 Service Worker 的代码后面加上 self.skipWaiting();
	  return self.skipWaiting();
	})
  );
});
```
不加 `self.skipWaiting()` 时，更新 Service Worker 文件，会看到如下图：
![](https://banzhuanmei.github.io/test-sw/images/waiting.png)
9:59:03 时更新的 Service Worker 处于 `waiting` 状态。

增加 `self.skipWaiting()` 后，刷新页面可以看到 10:00:44 更新的 Service Worker 被立马激活：

![](https://banzhuanmei.github.io/test-sw/images/skipwaiting.png)


# fetch，自定义请求响应
每次 Service Worker 控制的资源被请求时，都会触发`fetch`事件，监听 `fetch` 事件，处理 `response`。

```
// 此处 event 为 FetchEvent
self.addEventListener('fetch', event => {
// respondWith 劫持 http 响应，为页面的请求生成自定义的 response
  event.respondWith(
  // caches.match 通过匹配网络请求和 cache 里可获取资源的 url 和 vary header，确认是否可从缓存中获取资源
    caches.match(event.request).then(
	  (response) => {
	  // 缓存中可以匹配到时，直接返回匹配到的资源
	    if (response !== undefined) {
		   return response;
		 }
		 else { // 缓存中没有，请求后把资源放入缓存中
		   return fetch(event.request).then(
		     () => {
		     // 复制一份响应，放入名称为 CACHE_NAME 的缓存中
			    let responseClone = response.clone();
				 caches.open(CACHE_NAME).then(
				   (cache) => {
					  cache.put(event.request, responseClone);
					}
				 )
				 return response;
			  }
			).catch(() => { // 异常时，可返回备用的信息
			  return cache.match('/test-sw/images/waiting.png');
			})
		  }
		}
	  )
	)
});

```
`response.clone()`，这里为什么要克隆一份响应呢？因为请求和响应流只能被读取一次，克隆后的数据会发送到缓存中，原始响应数据返回给浏览器，这样都只读取了一次。

关于异常时，返回备用的数据，可试着把文件中用到的图片从缓存配置中去掉，在缓存配置中增加一个备用图片，断网刷新页面，会看到原图已经被备用图片代替：
![](https://banzhuanmei.github.io/test-sw/images/fetch-catch.jpg)

返回的 `response` 是异常时的备用图片：
![](https://banzhuanmei.github.io/test-sw/images/fetch-catch-response.png)

至此，Service Worker 的配置、注册、应用、管理有了一个大概完整的描述，其中参考了很多文章，很多小的知识点都需要学习，欢迎大家一起交流学习～～

# 参考文章

[https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers)
[https://developer.mozilla.org/zh-CN/docs/Web/API/Cache](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache)
[https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/)
[https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorkerContainer](https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorkerContainer)
[https://developer.mozilla.org/zh-CN/docs/Web/API/ExtendableEvent/waitUntil](https://developer.mozilla.org/zh-CN/docs/Web/API/ExtendableEvent/waitUntil)
[https://www.zhangxinxu.com/wordpress/2017/07/service-worker-cachestorage-offline-develop/](https://www.zhangxinxu.com/wordpress/2017/07/service-worker-cachestorage-offline-develop/)
[https://www.zhangxinxu.com/wordpress/2017/07/js-window-self/](https://www.zhangxinxu.com/wordpress/2017/07/js-window-self/)
[https://www.jianshu.com/p/0276db42dd2a](https://www.jianshu.com/p/0276db42dd2a)
[https://www.cnblogs.com/dojo-lzz/p/8047336.html](https://www.cnblogs.com/dojo-lzz/p/8047336.html)
[https://blog.csdn.net/ztguang/article/details/51224744](https://blog.csdn.net/ztguang/article/details/51224744)
