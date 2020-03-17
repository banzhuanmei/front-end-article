动画性能的终极指南——始终是测量

在这篇文章的后面，我们会看到性能最重要的部分：如何测量性能和分析瓶颈。没有数据，性能优化是没有意义的！我们将介绍如何使用 Chrome Devtools 分析性能。首先，一起看下 **Paint Flashing** 和 **Layer Borders** 两个工具。

Chrome 72 里面的功能——在较新的版本上，可能已经过时或被弃用。

## Paint Flashing 和 Layer Borders

**Paint Flashing** 用绿色框高亮显示页面需要重绘的部分。绘制通常是渲染过程中运行时间最长的步骤，因此要尽可能减少绘制。父元素样式改变流向子元素，或使用像 `top` 、 `left` 这样的布局属性，而不用类似 `transform` 的组合布局属性，都会引起不必要的绘制。

**Layer Borders** 用橙色框出页面上的合成图层。检测图层很重要，因为它们既能给性能带来好处，也能带来坏处。我们希望页面中的动画可以启动 GPU 硬件加速，图层由独立的合成器进行处理，但也不需要太多图层，因为每一个图层都需要内存和资源来管理。

译者注：GPU 硬件加速工作原理——渲染树中的每一个元素会被分配到一个图层中，使用 `transform` 的图层会由独立的合成器进程处理，所以不会在页面中重绘。

可以在 “More Tools” 的 **Rendering** tab 选项中找到它们。

![](https://static.viget.com/_600xAUTO_crop_center-center_none/1-ch-anim-perf3.png)

## 分析运行时性能

对于测量，最重要的是得出动画运行时的性能。我们想确保动画在所有设备上运行良好，从低端手机到高端台式机。无论用户的设备或连接质量如何，好的动画都可以无缝运行。

使用[此弹窗示例](https://codepen.io/chofsho/pen/oVgyXGhttps://codepen.io/chofsho/pen/oVgyXG)(稍微调整以适应本指南)完成本教程。首先我们分析未优化的版本。

### 开始

1.在 Chrome 中打开这个[链接](https://codepen.io/chofsho/pen/oVgyXG)，并打开 Devtools。

2.点击 **performance** 并确保 **Screenshots** 是被勾选的。点 **Capture Setting**(红色的齿轮icon)打开其他设置。

![](https://static.viget.com/_600xAUTO_crop_center-center_none/2-ch-anim-perf3.png)

3.通过减慢 CPU 模拟一个功能较弱的设备。这很有用，因为手机 CPU 比台式机弱得多。选择 **6x slowdown**。动画框现在应该明显变慢了。

![](https://static.viget.com/_600xAUTO_crop_center-center_none/3-ch-anim-perf3.png?mtime=20190329153223)

⭐️注：

* 如果动画没有明显变慢，试着在 Codepen 里添加更多框。性能可能因设备的处理能力而不同。
* 要得到最佳效果，请在隐身模式下打开 Chrome。隐身模式可以确保其他正在运行的任务(像 Chrome 扩展程序)不会影响测量。

### 拍一个快照

现在是时候录制动画了，这样我们就可以分析它的表现和可能包含不必要工作的部分。

1.点击 **Record** 按钮，运行几秒后点 **Stop**。

![](https://static.viget.com/_600xAUTO_crop_center-center_none/4-ch-anim-perf3.png?mtime=20190329153232)

2.结果如下！你会发现它看起来五颜六色，也许有些荒谬😳。这绝对是一件很重要的事情，所以让我们一步一步来看。

![](https://static.viget.com/_600xAUTO_crop_center-center_none/5-ch-anim-perf3.png?mtime=20190329153242)

### 分析结果

首先要了解的是，结果显示了动画如何随时间变化。如果把鼠标停在像截屏的区域，你会看到动画在该时间点的呈现：

![](https://static.viget.com/6-ch-anim-perf3.gif?mtime=20190329153251)

1.首先回顾动画的 FPS。动画必须达到每秒60帧的速度才能让用户感到流畅。看一下 **FPS 图表**，绿色条对应 FPS 的速度。绿色条越高，表示速度越快。绿色条上的红色条——意味着 FPS 太低以至于影响用户体验。另一个查看 FPS 的地方是 **Frames** 部分。你可以把鼠标停在每个绿色方块上查看特定帧的 FPS。

![](https://static.viget.com/_600xAUTO_crop_center-center_none/7-ch-anim-perf3.png?mtime=20190329153302)

2.接下来，看一下 **CPU** 的执行情况。如果 CPU 承担太多工作，可能会导致跳过动画帧。你可以在区域图或饼状图中查看：

![](https://static.viget.com/_600xAUTO_crop_center-center_none/8-ch-anim-perf3.png?mtime=20190329153342)

如果图标充满颜色，意味着 CPU 被最大化，我们需要最小化工作。颜色对应渲染过程的每一步：黄色代表 Javascript，紫色代表重算样式和布局，绿色代表绘制和复合图层。这里有很多紫色，意味着可能遇到[强制同步布局](https://www.viget.com/articles/animation-performance-101-optimizing-javascript/)。

3.现在，看一下 **Main** 部分，看看主线程上究竟发生了什么。你可以放大录制里的特定片段，以更好地查看活动。

![](https://static.viget.com/_600xAUTO_crop_center-center_none/9-ch-anim-perf3.png?mtime=20190329153353)

看“Animation Frame Fired” 事件——每当调用 `requestAnimationFrame()` 时，此事件都会被调用。堆栈条形图会告诉你触发事物的顺序。“Animation Frame Fired” 调用了回调 `movebox`，然后在 `movebox` 里触发了`boxes.forEach`。

`boxes.forEach` 接着触发了一堆紫色和浅褐色的事件。紫色事件顶部有红色三角——表明该事件存在问题。点击一个，可以在 **Summary** 中看到更多细节：

![](https://static.viget.com/_600xAUTO_crop_center-center_none/10-ch-anim-perf3.png?mtime=20190329153402)

这儿可以看到它是一个布局事件，并且可以看到强制重排的相应警告(a.k.a 强制布局)。看起来像是上面的 JS 强制执行的。注意它显示了“强制布局”并给出 **pen.js:41** 的链接——指向触发这个布局事件的代码行。

![](https://static.viget.com/_600xAUTO_crop_center-center_none/11-ch-anim-perf3.png?mtime=20190329153414)

这一行是 `box.getBoundingClientRect()`。同样我们知道这会强制布局，因为在第 37 行修改 DOM 之后，第41行正在读取 DOM。

理想情况下，浏览器希望在 JavaScript 完成之后执行所有渲染事件(紫色部分)。没有强制布局的动画看起来是这样的，黄色下面没有紫色：

![](https://static.viget.com/_528xAUTO_crop_center-center_none/15-ch-anim-perf3.png?mtime=20190329153920)

这个分析中，能看出来主要的瓶颈是 JavaScript 强制同步布局导致的。它创造了不必要的工作并使 CPU 最大化。带着这个观点，我们分析一下优化版，看看有什么不同。

### 优化版

下面展示了这个版本运行时性能的记录：

![](https://static.viget.com/_600xAUTO_crop_center-center_none/12-ch-anim-perf3.png?mtime=20190329153435)

比起未优化的版本，可以立马看到优化版有多好。首先，我们实现了一致高的 FPS，并且 FPS 上面没有红色条。其次，CPU 图表没有填满颜色，渲染工作比之前的少了很多。放大主线程部分，也会发现不再有强制布局：

![](https://static.viget.com/_600xAUTO_crop_center-center_none/13-ch-anim-perf3.png?mtime=20190329153445)

黄色条、红色条或红色三角下面不再有紫色，这是通过采取以下优化实现的：

1. 在 DOM 写入之前执行所有 DOM 读取。
2. 缓存任何可以提前保存的 DOM 查询。
3. 用动画 `transform` 代替 `margin-left`。

让我们以一个显示上面第三点重要性的图表结束——使用触发复合图层的属性代替布局属性。左边图表显示使用了 `margin-left` 的弹跳方块动画，右边图表是使用 `transfrom`。你可以看到修改一行代码，减少了多少绘制时间！

<div style="float:left;width:49%;">
	<img src="https://static.viget.com/_540xAUTO_crop_center-center_none/14-left-ch-anim-perf3.png?mtime=20190329153505" />
	<i>margin-left</i>
</div>

<div style="float:right;width:49%;">
	<img src="https://static.viget.com/_540xAUTO_crop_center-center_none/14-right-ch-anim-perf3.png?mtime=20190329153517" />
	<i>transform</i>
</div>

## 结束语

提升动画性能不只是知道使用什么 CSS 属性——认为只是使用 `transform` 和 `opacity` 或 `translateZ` 就能修复性能。想要动画性能流畅，理解原因和方法很重要——浏览器如何渲染，为什么重要，如何测量优化。

希望这篇文章有助于你学到这些知识。

>原文链接：https://www.viget.com/articles/animation-performance-101-measuring-with-dev-tools/
