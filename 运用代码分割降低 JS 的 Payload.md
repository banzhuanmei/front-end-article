* 现在很多网站都把 JavaScript 打包成一个单独的大文件。这样提供 JavaScript 文件时，下载和执行时间对移动设备和网络来说都很重要。
* 可以用代码切分代替大的打包文件，即把 JavaScript 文件分成更小的块。通过发送需要提前加载的最少代码，来提升页面加载时间。剩余的可以按需加载。
* 你需要代码拆分吗？用 Lighthouse 的 [JS启动时间过长](https://developers.google.com/web/tools/lighthouse/audits/bootup) 和 DevTools 的 [代码覆盖面版](https://developers.google.com/web/updates/2017/04/devtools-release-notes#coverage) 测量 app 的脚本对性能影响和未使用的脚本。
* 可以通过以下几种方式完成代码拆分：
* **供应商拆分** 可以把供应商代码(例如：React 或 loadsh)和应分开。当供应商代码或应用代码被更改时，供应商拆分会隔离缓存失效对响应的影响。这是每个 app 都应该有的。
* **入口点拆分**通过在 app 入口处分离代码，在 webpack 和 Parcel 等工具在构建依赖时启动。这最适合未使用客户端路由的页面或应用，或部分使用服务端路由、部分是单页面应用的混合应用。
* **动态拆分**把 import() 引入的代码分离，最适合单页面应用。
* 尽可能选择拆分代码的工具([Preact CLI](https://github.com/developit/preact-cli/),[Gatsby](https://www.gatsbyjs.org/),[PWA Starter Kit](https://github.com/Polymer/pwa-starter-kit/)等)。[React](https://reactjs.org/docs/code-splitting.html),[Vue](https://vuejsdevelopers.com/2017/07/03/vue-js-code-splitting-webpack/) 和 [Angular](https://angular.io/guide/lazy-loading-ngmodules) 都支持手动代码拆分。

也许你以前听过这些，但在中等移动设备上，[页面如果有很多 JS](https://httparchive.org/reports/state-of-javascript#bytesJs) [不是一件好事情](https://speedcurve.com/blog/your-javascript-hurts/)。然而，对过多的 JS 做限制不是最好的方式。每个应用程序不同，其中 JS 有的多，有的少。用户和设备各不相同！

这就是考虑如何提供 JS 的原因。你是不是把所有 JS 打包成一个大文件放到所有页面中？如果是的话，你会考虑如何用代码分割来帮到你！

## 太多太快

许多 app 把脚本放在一个文件里，在初始加载时提供一个大的打包文件。这个文件不止支持初始路由，还支持每个路由中的每个交互——无论路由是否被访问！

全有-或-全无的方式效率较低。加载、解析、执行未使用的代码会延长 app [交互时间(TTI)](https://developers.google.com/web/tools/lighthouse/audits/time-to-interactive)，这意味着强制用户在使用之前等待不必要的资源。移动设备的用户可能更会感觉到这个问题，移动设备较慢的处理器或网络可能导致进一步的延迟。下图展示了移动设备和处理器更好的电脑的解析、编译时间：

![](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/code-splitting/images/figure-1-1x.png)

图1显示移动设上 JS 启动的时间。来源：[Addy Osmani](https://twitter.com/addyosmani) 的 [JS 启动性能](https://medium.com/reloading/javascript-start-up-performance-69200f43b201)。

我们都喜欢用较快的 app，[有很多关于提升不同指标的的案例](https://wpostats.com/)。跟全有-或-全无相比，代码拆分更注重打包当前路由需要的最少代码，而不是一次加载所有。

## 我需要代码拆分吗？

我需要拆分我们 app 的代码吗？这是一个有效的问题，经常回答者都说：视情况而定。为了回答这个问题，需要依赖你对 app 架构、JS 加载、[Lighthouse](https://developers.google.com/web/tools/lighthouse/) 和 DevTools 的理解。

对于性能新手，Lighthouse 需要的成本最小。在 Chrome 中打开 app 的初始页面，通过Devtools 的 Audits 面板打开 Lighthouse。你需要关注 [JS 启动时间过长](https://developers.google.com/web/tools/lighthouse/audits/bootup)这项审核，它标记了 JS 明显延迟了 app 的交互时间。

![](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/code-splitting/images/figure-2-1x.png)

图2中 JS 的启动时间显示了执行较多的脚本。

幸运的是，可以通过审核收集的信息和 DevTools 中检查代码覆盖的工具(DevTools聚焦时，用 esc 键打开)查出当前路由未使用的代码。

![](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/code-splitting/images/figure-3-1x.png)

图3 DevTools 的代码覆盖面板显示了当前页面中有多少 JS 代码被使用。

`注：即使你对 app 的代码进行了拆分，页面中仍可能有未使用的代码。[Tree shaking](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/tree-shaking/) 可能能使包更小！`

如果你目前的 app 没有花费过多的时间用来交互或下载，你可能不需要使用代码拆分，如果不是，请继续往下看！

## 开始使用代码拆分

没有实例只谈论代码拆分可能会让读者更困惑。为了更明确，这篇文章用一个示例 app 展示不同的代码拆分方法，你可以作为参考。

`注：这篇文章覆盖了一些示例 app 拆分代码的技巧(例如基于哈希输出文件名的版本控制和使用 [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin))。`

![](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/tree-shaking/images/figure-3-1x.png)

图4中的示例 app，能搜到吉他踏板的数据。

它有三个路由：

1. 搜索页面，是用户搜索[吉他踏板](https://en.wikipedia.org/wiki/Effects_unit#Stompboxes)的默认地址。
2. 踏板详情页面，在用户点击搜索结果时展示，也可以在当前页收藏喜欢的踏板。
3. 喜欢的踏板页面，展示用户喜欢的踏板。

大多数例子会展示用 webpack 的不同方式按路由拆分代码，但动态拆分代码部分会展示如何使用 Parcel 拆分。一起看下如何通过 webpack 入口点拆分代码。

### 通过入口点拆分代码

[入口点](https://webpack.js.org/concepts/entry-points/)是 webpack 分析 app 依赖的文件。资源、路由、功能像是 app 这棵树的枝干。有的 app 只有一个入口点，有的有[多个入口点](https://webpack.js.org/concepts/entry-points/#multi-page-application)。

什么时候这种方式是有效的呢：非单页面应用(SPA)的 app，或部分不是由客户端渲染、部分是 SPA 的混合应用。

需要注意的是：如果入口点共用第三方文件或模块，脚本中可能会出现重复的代码。马上为大家介绍这个陷阱。

示例 app 中的 index.js、detail.js 和 favorites.js 对应前面描述的每个路由。这三个脚本文件包含 [Preact](https://preactjs.com/) 组件来渲染路由对应的页面。

`注：如果你想查看基于入口点拆分实现的示例代码库，可以访问 app 在 [webpack-entry-point-splitting](https://github.com/malchata/code-splitting-example/tree/webpack-entry-point-splitting) 分支的代码`

#### 配置 webpack 多入口拆分代码

在 webpack 中，可以通过在 [entry 的配置项](https://webpack.js.org/configuration/entry-context/#entry)中定义拆分点：

```
module.exports = {
  // ...
  entry: {
    main: path.join(__dirname, "src", "index.js"),
    detail: path.join(__dirname, "src", "detail.js"),
    favorites: path.join(__dirname, "src", "favorites.js")
  },
  // ...
};
```

这种方式的好处是，当有多个入口点时，webpack 会把它们视为单独的依赖，代码会以块命名并被自动拆分：
```
 Asset       Size  Chunks             Chunk Names
js/favorites.15793084.js   37.1 KiB       0  [emitted]  favorites
   js/detail.47980e29.js   44.8 KiB       1  [emitted]  detail
     js/main.7ce05625.js   49.4 KiB       2  [emitted]  main
              index.html  955 bytes          [emitted]
             detail.html  957 bytes          [emitted]
          favorites.html  960 bytes          [emitted]
```
可能你已经猜到，块命名来自 entry 的配置项，它指明了块包含的代码和对应的页面。这个 app 还使用了 html-webpack-plugin 生成每个页面对应的块文件。

#### 删除重复代码

尽管我们给每个页面都很好地拆分了代码，但仍然有问题：每个块之间有很多重复的代码。因为 webpack 把入口配置作为依赖，并不会评估块之间共用的代码。如果打开 [webpack 中的 source maps](https://webpack.js.org/configuration/devtool/)，用像 [Bundle Buddy](https://github.com/samccone/bundle-buddy) 或 [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer) 这样的工具分析代码，会看到所有脚本中有多少重复代码。

![](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/code-splitting/images/figure-5-1x.png)

图5 Bundle Buddy 显示打包文件间有多少共用代码。

重复的代码是第三方脚本。为了解决这个问题，可以配置 [optimization.splitChunks](https://webpack.js.org/plugins/split-chunks-plugin/#optimization-splitchunks) 给共用代码创建一个单独的块：

```
module.exports = {
  // ...
  optimization: {
    splitChunks: {
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/i,
          name: "vendors",
          chunks: "all"
        }
      }
    },
    runtimeChunk: {
        name: "vendors"
    }
   },
  // ...
};

```
此配置项表示要把从 node_modules 中加载的脚本输出成一个以第三方名称命名的单独块。它能生效的原因是，我们所有第三方脚本都是通过 npm 安装到 node_modules中，配置项中 [test ](https://webpack.js.org/plugins/split-chunks-plugin/#splitchunks-cachegroups-test)可以检测到。[runtimeChunk 配置项](https://webpack.js.org/configuration/optimization/#optimization-runtimechunk)也指定把 [webpack 的运行时](href="https://webpack.js.org/concepts/manifest/#runtime)移动到第三方块中，避免应用代码里有重复代码。添加完这些配置，重新 rebuild 项目，输出项显示已经把第三方代码移到单独文件中：

```
Asset      Size  Chunks             Chunk Names
  js/vendors.4080088b.js    38 KiB       0  [emitted]  vendors
   js/detail.a9849678.js  7.52 KiB       2  [emitted]  detail
     js/main.a5244634.js  12.1 KiB       3  [emitted]  main
js/favorites.17826216.js  6.23 KiB       1  [emitted]  favorites
              index.html  1.01 KiB          [emitted]
             detail.html  1.01 KiB          [emitted]
          favorites.html  1.01 KiB          [emitted]
          
```
由于第三方脚本和共用代码已经被拆成专门的块，减少了入口脚本的大小， Bundle Buddy 为我们呈现了一个更好的结果：

![](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/code-splitting/images/figure-6-1x.png)

图6显示了块之间的 input lines 和共用代码明显减少。

拆分第三方代码前，每个包之间大概有几千行共用代码，现在明显减少了。虽然把第三方代码分离出来会增加额外的 HTTP 请求，但如果你的 app 仍然在 HTTP/1 上，这可能只是一个问题。还有，这种方式提供脚本更适合缓存。如果你有一个很大的打包文件，无论应用程序和第三方代码哪个改变，都需要再下一次完整的包。

`注：不要忘了给返回用户设置[最佳缓存策略](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)`

虽然这种方式的拆分代码很有用，但它并不是唯一可用的方法，接下来介绍动态代码拆分！

### 动态拆分代码

入口点拆分代码是合理且直观的，但对你的 app 来说它可能不是最好或最实用的。另一种方式是使用[动态的 import() 语句](https://developers.google.com/web/updates/2017/11/dynamic-import)有效地延迟加载脚本：

```
import("./myFancyModule.js").then(module => {
  module.default(); // Call a module's default export
  module.andAnotherThing(); // Call a module's named export
});

```

由于 import() 返回一个 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象，你还可以使用 `async`/ `await`。

```
let module = await import("./myFancyModule.js");
module.default(); // Access a module's default export
module.andAnotherThing(); // Access a module's named export

```

无论你更喜欢哪种方式，Parcel 和 webpack 都可以检测到 import() 语句，并把导入的进行拆分。

什么时候这种方式比较有用呢：假如你正开发的是一个单页面应用，并且包含许多分散的、用户不一定能用到的功能，延迟加载这些就可以减少 JS 解析/编译活动和网络传输字节数。

需要注意的事项：动态导入脚本会启动网络请求，这意味着操作可能会被延迟。马上介绍一些缓解这种情况的方法。

首先，介绍下 Parcel 中动态拆分代码的原理。

#### 用 Parcel 动态拆分代码

[Parcel](https://parceljs.org/) 是动态拆分代码最简单的工具。不需要任何配置，Parcel 构建了一个依赖树，用来计算静态和动态的模块，并输出和输入时的文件名一样的脚本。

`如果你想了解示例 app 是怎么使用 Parcel 动态拆分代码的，可以在 [parcel-dynamic-splitting](https://github.com/malchata/code-splitting-example/tree/parcel-dynamic-splitting) 分支上查看。`

在这个版本的示例 app 中，是由[preact-router]() 和 [preact-async-route]() 提供客户端路由。如果没有动态导入的模块，所有路由需要的组件都需要提前导入，并由客户端下载：

```
import Router from "preact-router";
import { h, render, Component } from "preact";
import Search from "./components/Search/Search";
import PedalDetail from "./components/PedalDetail/PedalDetail";
import Favorites from "./components/Favorites/Favorites";

render(<Router>
  <Search path="/" default/>
  <PedalDetail path="/pedal/:id"/>
  <Favorites path="/favorites"/>
</Router>, document.getElementById("app"));

```
这样无论用户是否访问，都加载出每个路由对应的组件，会错过用懒加载提升性能的机会。在这个例子中，用 import() 和 preact-async-route 延迟路由 /pedal/:id 和 /favorites 对应的组件，代码如下：

```
import Router from "preact-router";
import AsyncRoute from "preact-async-route";
import { h, render, Component } from "preact";
import Search from "./components/Search/Search";

render(<Router>
  <Search path="/" default/>
  <AsyncRoute path="/pedal/:id" getComponent={() => import("./components/PedalDetail/PedalDetail").then(module => module.default)}/>
  <AsyncRoute path="/favorites" getComponent={() => import("./components/Favorites/Favorites").then(module => module.default)}/>
</Router>, document.getElementById("app"));

```
跟之前的例子相比，你会发祥有几处不同：

1.只静态导入了 Search 组件，因为它是默认路由对应的组件，所以预先加载它是合理的。

2.preact-async-route 通过 AsyncRoute 组件处理异步路由。

3.PedalDetail 和 Favorites 组件在用户访问对应地址时，AsyncRoute 组件通过 import() 语句把它们延迟加载进来。

build app，Parcel 会输出如下：

```
dist/src.e54c18ce.js             65.56 KB    2.87s
dist/PedalDetail.7f417dfd.js      7.66 KB    1.52s
dist/Favorites.02c87dad.js        3.06 KB    1.17s
dist/index.html                     964 B    882ms

```
Parcel 在另配置的情况下，可以自动把动态导入的脚本拆分成按需延迟加载的块。

`警告：这个例子中没有使用第三方代码！`

访问默认路由时，只需加载默认路由对应的脚本。当用户访问踏板细节或收藏地址时，它们的脚本也会按需加载。

##### 用 webpack 动态拆分代码

像 Parcel 一样，webpack 在遇到 import() 时会把动态导入的代码拆成单独的文件，但它没有像 Parcel 一样输出带文件名的文件：

```
Asset       Size  Chunks             Chunk Names
      js/0.98b97315.js  349 bytes       0  [emitted]
      js/2.185c8ab6.js   6.84 KiB       2  [emitted]
js/vendors.85016c2f.js     45 KiB       3  [emitted]  vendors
      js/4.7e6bc0c9.js  528 bytes       4  [emitted]
   js/main.4ea5c500.js   12.3 KiB       5  [emitted]  main
      js/1.e8943ee4.js   3.42 KiB       1  [emitted]
            index.html   1.01 KiB          [emitted]

```
能看到 webpack 给导入的文件命名为数字，而不是名字。这对用户来说没什么关系，但对开发者来说却是个问题。为了解决这个问题，需要用一种称为内联指令的特殊注释，告诉 webpack 应该以什么名字输出文件：

```
render(<Router>
  <Search path="/" default/>
  <AsyncRoute path="/pedal/:id" getComponent={() => import(/* webpackChunkName: "PedalDetail" */ "./components/PedalDetail/PedalDetail").then(module => module.default)}/>
  <AsyncRoute path="/favorites" getComponent={() => import(/* webpackChunkName: "Favorites" */ "./components/Favorites/Favorites").then(module => module.default)}/>
</Router>, document.getElementById("app"));

```
其中，内联指令 webpackChunkName 指 wenpack 应该输出的块名。使用后，webpack 输出了理想的块命名：

```
Asset       Size  Chunks             Chunk Names
    js/simpleSort.5feb0579.js  358 bytes       0  [emitted]  simpleSort
   js/PedalDetail.c5c9b3bd.js   6.85 KiB       2  [emitted]  PedalDetail
       js/vendors.311ae574.js   45.1 KiB       3  [emitted]  vendors
js/toggleFavorite.b8d7feed.js  541 bytes       4  [emitted]  toggleFavorite
          js/main.0b31627f.js   12.3 KiB       5  [emitted]  main
     js/Favorites.0c1db0a6.js   3.43 KiB       1  [emitted]  Favorites
                   index.html   1.01 KiB          [emitted]

```
这种方式有些笨拙，它意味着你需要先付出努力才能确保块有一个有意义的名字，但它确实做到了！如果你想了解示例 app 是怎么使用 webpack 拆分动态代码的，可以去 [webpack-dynamic-splitting](https://github.com/malchata/code-splitting-example/tree/webpack-dynamic-splitting) 分支查看。

## 加载性能的考虑因素

代码拆分的一个潜在痛点是增加了脚本的请求量，即使在 HTTP/2 中也会有挑战。一起看下使用代码拆分时如何提高加载性能。

### 利用 service worker 预缓存脚本

PRPL 模式的 Ps 代表预缓存，它在加载初始路由后用 service worker 预缓存剩余路由和功能。以下方式使预缓存非常有效：

1.它不会影响初始加载的性能，因为 service worker 注册和后续的预加载发生在 app 加载之后。

2.service worker 预缓存剩余路由和功能确保在后面请求时立即可用。

当然，由于很多原因，把 service worker 添加到用工具生成代码的 app 中会有挑战性，例如输出命名带有哈希值的文件。幸运的是 [Workbox](https://developers.google.com/web/tools/workbox/) 有一个插件可以很轻松地为你的 app 生成 service worker。至少你可以安装 workbox-webpack-plugin 并在 webpack 中做如下配置：

```
const { GenerateSW } = require("workbox-webpack-plugin");

```
然后可以把 `GenerateSW` 加到 `plugins` 的配置中：

```
module.exports = {
  // ...
  plugins: [
    // ... other plugins omitted
    new GenerateSW()
  ]
  // ...
};

```
有了这个配置，Workbox 会生成一个 service worker 把 app 所有的 JS 都预缓存。对小应用来说可能是好的，但大 app 的话，你可能会想限制预缓存的内容，可以把它们加到 chunks 的配置中：

```
module.exports = {
  // ...
  plugins: [
    new GenerateSW({
      chunks: ["main", "Favorites", "PedalDetail", "vendors"]
    })
  ]
  // ...
};

```
使用白名单的方式，可以确保 service worker 只预缓存你想要的脚本。可以在 [webpack-dynamic-splitting-precache](https://github.com/malchata/code-splitting-example/tree/webpack-dynamic-splitting-precache) 分支上查看如何在 app 中使用 Workbox。

### 预取预加载脚本

service worker 的预缓存路由和功能是提升 app 加载性能的一种方式，但 service work 应该被视为渐进式增强。如果没有它们(或其他策略)，你可能需要考虑预取或预加载块。
添加 `rel=prefetch` 和 `rel=preload` 都会在浏览器之前获取指定资源，通过屏蔽延迟提高性能。乍一看它两很相似，但表现却很不一样：

1. [`rel=prefetch`](https://www.w3.org/TR/resource-hints/#prefetch) 是对另一个路由中的非关键资源低优先获取。当浏览器空闲时，由 `rel=prefetch` 启动请求。

2. [`rel=preload`](https://www.w3.org/TR/preload/) 是对当前路由的关键资源高优先获取。`rel=preload` 发起的资源请求可能比浏览器更早，但预加载非常细微，你可能会想看看[指南](https://developers.google.com/web/fundamentals/performance/resource-prioritization#preload)和[规范](https://www.w3.org/TR/preload/)。

如果你想深入了解这些资源提示，可以阅读这篇文章。出于这篇文章的主旨，我会少讲一些资源提示的内容，因为要在 webpack 里面使用它。

#### 预获取

为确定会访问或使用的路由和功能预取脚本可能是合理的，但还没这么做。这篇文章中，在入口文件 index.js 中给 app 挂载 Router 组件，是预取的一个好的例子：

```
render(<Router>
  <Search path="/" default/>
  <AsyncRoute path="/pedal/:id" getComponent={() => import(/* webpackChunkName: "PedalDetail" */ "./components/PedalDetail/PedalDetail").then(module => module.default)}/>
  <AsyncRoute path="/favorites" getComponent={() => import(/* webpackPrefetch: true, webpackChunkName: "Favorites" */ "./components/Favorites/Favorites").then(module => module.default)}/>
</Router>, document.getElementById("app"));

```
给收藏页面的 AsyncRoute 添加 webpackPrefetch，如果没有预取这个路由的脚本，用户请求时可能会有如下延迟：

![](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/code-splitting/images/figure-7-1x.png)

图7.在受限连接时(慢速3G)请求收藏页面的脚本。

慢速时，用户必须等几秒，才能收到收藏页面的脚本。 但使用 `webpackPrefetch` 的话，首次加载 app 时就会减少等待时间，并会预取 JS。

![](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/code-splitting/images/figure-8-1x.png)

图8.初始路由加载后就预取收藏页的脚本，当用户请求时，浏览器会立马从缓存中取出文件。

预取通常有低风险，因为它是在空闲时间获取资源的，不会很明显地抢占宽带。也就是说还是可能会浪费宽带，所以你得确保预取出来的东西都会被合理使用。

`注：可以在 [webpack-dynamic-splitting-prefetch](https://github.com/malchata/code-splitting-example/tree/webpack-dynamic-splitting-prefetch) 分支上查看预取是如何生效的`

#### 预加载

预加载类似于预取，但还是不同。`webpackPreload` 可以像 `webpackPrefetch` 唤起预取一样唤起预加载。然而在我经验里，用 `webpackPreload` 预加载动态引入的资源，和把路由对应的所有功能打包成一个块大致一样。

我认为，预加载对渲染初始路由的关键资源比较有效。 Twitter 这样做是为了加快 [Twitter Lite](https://mobile.twitter.com/home) 的加载。

![](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/code-splitting/images/figure-9-1x.png)

图9.Devtools 中的 Twitter Lite DOM 快照显示的几个预加载 JS 资源。

遗憾的是，webpackPreload 只适用于 import()，为了预加载示例 app 初始路由的关键块，需要依赖 [preload-webpack-plugin](https://github.com/GoogleChromeLabs/preload-webpack-plugin)。安装插件后，做如下配置：

```
const PreloadWebpackPlugin = require("preload-webpack-plugin");

```
然后在 plugins 数组中添加 PreloadWebpackPlugin 的实例，预加载 main、vendors 块。

```
plugins: [
  // Other plugins omitted...
  new PreloadWebpackPlugin({
    rel: "preload",
    include: ["main", "vendors"]
  })
]

```

此配置通过 `<link>` 给 main块、vendors块增加预加载。

![](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/code-splitting/images/figure-10-1x.png)

可以在 Devtools 看出来，预加载已经加在 `main` 和 `venders` 的 `<head>` 中。

虽然预加载没有给示例 app 提供很大的性能提升，但它提升了 app 的加载性能，因为有许多 JS 和其他资源会争宽带。可以在 [webpack-dynamic-splitting-preload](https://github.com/malchata/code-splitting-example/tree/webpack-dynamic-splitting-preload) 分支查看预加载在示例 app 的应用。

`注：preload-webpack-plugin 必须与 html-webpack-plugin 一起使用！添加 preload-webpack-plugin 时，必须把它放在 html-webpack-plugin 最后一个实例的后面。`

## 结论和资源

毫无疑问，代码拆分是很难的。更重的是，如何在特定的 app 中拆分代码需要你花时间研究。如果你想了解更多关于代码拆分或其他这个主题的内容，可以看下面的文章列表：

* [Official webpack code splitting docs.](https://webpack.js.org/guides/code-splitting/)
* [Official Parcel.js code splitting docs.](https://parceljs.org/code_splitting.html)
* [Official React code splitting docs.](https://reactjs.org/docs/code-splitting.html)
* [Official Vue code splitting docs.](https://vuejsdevelopers.com/2017/07/03/vue-js-code-splitting-webpack/)
* [Official Angular code splitting docs.](https://angular.io/guide/lazy-loading-ngmodules)
* [Dynamic import() guidance here on Web Fundamentals.](https://developers.google.com/web/updates/2017/11/dynamic-import)

提升 app 性能会获得回报，因为用户会发现你的 app 更好用！










