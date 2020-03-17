Web workers，service worker，和 worklets，我把它们都称作 “Javascript Workers”，虽然它们在工作方式上确实有一些相似之处，但它们的用途几乎没有重叠。

笼统来讲，worker 是一个脚本，它运行在独立于浏览器主线程的线程上。如果你想通过 `<script>` 标签把典型的 JS 文件放入 HTML 中，脚本是运行在主线程的。如果主线程中有太多活动，它会降低网站速度，使交互紧张，响应缓慢。

Web workers，service workers 和 worklets 都是运行在一个独立线程上的脚本。那么这三种 workers 的区别是什么呢？

## Web workers

Web workers 是最通用的 worker 类型。跟接下来看到的 service workers 和 worklets 不同，Web workers 没有特定的用例，除了和主线程分开运行的功能。因此，web workers 可以用来卸载主线程中任何繁重的处理(译者注：此处卸载是指分担主线程中操作运算比较重的工作)。

![](https://bitsofco.de/content/images/2018/11/web-worker.jpg)

Web workers 是使用 `Web Workers API` 创建的。为我们的 worker 创建一个专用的 JS 文件，然后可以把它添加为新的 **Worker**。

```
/* main.js */

const myWorker = new Worker('worker.js');

```
这会开始运行 worker.js 中的任何代码。正如我提到的，它几乎可以是任何东西，但 web workers 对卸载需要很长时间的进程，或与其他进程并行的进程最有用。一个特别好的例子是图像处理 web 应用程序 [Squoosh](https://squoosh.app)，它用 web workers 处理图像操作的任务，使主线程可供用户和应用程序交互，而不会中断。

和所有 workers 一样，web workers 没有权限访问 DOM，意味着任何需要的信息在 worker 和 main.js 之间传递时都必须使用 **window.postMessage()**。

```
/* main.js */

// 创建 worker
const myWorker = new Worker('worker.js');

// 主线程向 worker 发送消息
myWorker.postMessage('Hello!');

// 主线程通过 onmessage 指定监听函数，接收 worker 中的消息
myWorker.onmessage = function(e) {
  console.log(e.data);
}

```

在我们 worker 脚本中，可以监听主脚本的消息，并返回一个响应。

```
/* worker.js */

// 从 main.js 中接收信息
self.onmessage = function(e) {
  console.log(e.data);

  // 给主脚本文件发送消息
  self.postMessage(workerResult);
}

```

## Service workers

Service workers 是一种在浏览器和网络或缓存之间作为代理的 worker。

![](https://bitsofco.de/content/images/2018/11/service-worker.jpg)

像 web workers 一样，service workers 在 main.js 中注册，引用一个专门的 service worker 文件。

```
/* main.js */

navigator.serviceWorker.register('/service-worker.js');

```
与常规的 web workers 不一样，service workers 有一些额外的功能，使它可以完成代理功能。一旦它们被安装激活，service workers 可以拦截 main.js 中的任何网络请求。

```
/* service-worker.js */

// 安装
self.addEventListener('install', function(event) {
    // ...
});

// 激活
self.addEventListener('activate', function(event) {
    // ...
});

// 监听 main.js 中的网络请求
self.addEventListener('fetch', function(event) {
    // ...
});

```
一旦被拦截，service worker 可以通过缓存返回内容，而不用通过网络来响应，从而允许 web 应用在离线状态下运行！

```
/* service-worker.js */

self.addEventListener('fetch', function(event) {
    // 从缓存中返回数据
    event.respondWith(
        caches.match(event.request);
    );
});

```

## Worklets

Worklets 是一个非常轻量，高度特定(译者注：此处指比较灵活，可定制性强)的 worker。它们可以使开发人员看到浏览器渲染过程的各个部分。

渲染页面时，浏览器经历了很多步骤。我在[“理解关键渲染路径”](https://bitsofco.de/understanding-the-critical-rendering-path/)的文章中更详细地介绍了这一点，但这里我们需要关注四个步骤——样式，布局，绘制和组合。

![](https://bitsofco.de/content/images/2018/11/frame-full.jpg)

<center>*图片来源：Paul Lewis 的[渲染性能](https://developers.google.com/web/fundamentals/performance/rendering/)*</center>

一起看下 Paint 阶段。这个阶段浏览器把样式应用在每个元素身上。worklet 在这个阶段的钩子是 [Paint Worklet](https://www.w3.org/TR/css-paint-api-1/)。

Paint Worklet 允许我们创建自定义图片，并且可以应用在 CSS 期望的任何位置，例如定义 **background-image** 的属性值。

要创建一个 worklet，跟所有 worker 一样，我们要在 main.js 中注册它，引用专门的 worklet 文件。

```
/* main.js */

CSS.paintWorklet.addModule('myWorklet.js');

```

在 worklet 文件中，我们可以创建一个自定义图片。**paint** 方法和 Canvas Api 很相似。下面是一个简单的黑白渐变的例子。

```
/* myWorklet.js */

registerPaint('myGradient', class {
  paint(ctx, size, properties) {
    var gradient = ctx.createLinearGradient(0, 0, 0, size.height / 3);

    gradient.addColorStop(0, "black");
    gradient.addColorStop(0.7, "rgb(210, 210, 210)");
    gradient.addColorStop(0.8, "rgb(230, 230, 230)");
    gradient.addColorStop(1, "white");

    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, size.width, size.height / 3);
  }
});

```
最后，我们可以在 CSS 中使用这个新 worklet，创建的自定义图片将会像任何其他背景图一样被应用。

```
div {
    background-image: paint(myGradient);
}

```
除了 Paint Worklet 之外，还有其他 worklets 的钩子关联渲染过程的其他阶段。[Animation Worklet](https://wicg.github.io/animation-worklet/) 挂钩组合阶段，[Layout Worklet](https://drafts.css-houdini.org/css-layout-api-1/#layout-worklet) 挂钩布局阶段。

## 简要概括

回顾一下，web workers，service workers 和 worklets 都是在独立于浏览器主线程的线程上运行的脚本。不同之处是它们使用的位置和实现这些用例具有的功能。

**Worklets** 是浏览器渲染过程的钩子，使我们能对浏览器的渲染过程(如样式和布局)进行低级访问。

**Service workers** 是浏览器和网络间的代理。通过拦截文件发出的请求，service workers 可以把请求重定向到缓存，从而实现离线访问。

**Web workers** 是通用脚本，使我们能从主线程中卸载处理器密集型的工作。
