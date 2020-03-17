# 使用 auto-fit 与 minmax 实现 Grid 瓦片布局

![](https://gedd.ski/img/tile-layouts.jpg)

瓦片布局可能是前端人员构建的最常见的布局。尤其是在内容领域。由于 CSS Grid 带来了新的 **minmax** 和 **auto-fit**，平铺布局从未如此简单地被创建。Lemme 展示了使用 CSS Grid 制作完美的响应式瓦片布局。

## 瓦片布局无处不在

你可能意识到了一些，**Google 音乐** 中几乎所有页面都是平铺布局：

![](https://gedd.ski/img/new-releases.jpg)

旁注：只是我还是许多新发布比较糟糕？像它们刚刚放弃了一样，我指的是其中一个乐队的名字字面意思是垃圾。

无论如何，许多博客像我这样以平铺的方式呈现帖子：

![](https://gedd.ski/img/blog-tiles.jpg)

你可能从未见过 Twitter 时刻页面，因为它毫无价值，但它也只是一个平铺布局，其中一些网格项跨越更多单元格：

![](https://gedd.ski/img/twitter-moments.jpg)

你明白了，瓦片布局无处不在。

## auto-fit 就是为瓦片布局产生的

我们使用新的 **auto-fit** 关键字，创建瓦片布局。它使 Grid 创建尽可能多的给定大小的列，以适应网格容器的空间。

所以，我们想把 Grid 分成尽可能多的宽为 **200px** 的列，以适应空间：

```
.app {
  display: grid;
  grid-gap: 15px;
  grid-template-columns: repeat(auto-fit, 200px);
}
```

![](https://gedd.ski/img/auto-fit-200.jpg)

在宽为 **1400px** 的 div 中，有足够的空间来创建六个宽为 **200px** 的列，每个列之间有 **15px** 的间距。

此时，你可以认为你的瓦片布局足够好，并把它展示出来，它会跟网上的很多瓦片布局一样好。但看看我们网格的右侧——有一片看起来不太好的大空白——这里不够放下另一列。

CSS Grid 非常适合使这种瓦片布局更好。

## minmax() 支持你

可以调整我们的瓦片布局，使瓦片充满 Grid 的所有空间，而不是在最后留下一块空白。

我们通过使用 **minmax** 创建弹性的列来实现不留空白的目标。

```
.app {
  display: grid;
  grid-column-gap: 15px;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}
```

![](https://gedd.ski/img/minmax-tiles.jpg)

不再浪费右边的空间了！

**minmax() 是如何做到的呢？**

**minmax(200px, 1fr)** 使列以最小尺寸 **200px** 开始，所以 Grid 创建了尽可能多的宽为 **200px** 的列——六个，像以前一样在右侧留下大约 **130px** 的空间。但这次，Grid 将额外的空间均匀分配到六列，因为它们的最大尺寸设置为 **1fr**(一部分自由空间，译者注：`1fr` 表示让浏览器将网格空间进行均分，每列占一份)。

**auto-fit** 和 **minmax** 的结合使瓦片布局既灵活又可以占满空间。

## 响应准备

最好的部分是——这种方式的瓦片布局已经适合在较小的移动屏幕上使用。

![](https://gedd.ski/gif/tiles.gif)

当空间不够放宽为 **200px** 的列时，Grid 会自动创建一个新行，把列放到下一行。**minmax** 的最大值为 **1fr** 确保列总会填满整行。这只是 CSS Grid 对[响应式布局](https://gedd.ski/post/grid-for-responsive-layout/)非常有用的原因之一。

## 带有瓦片模式的固定布局

现在你了解了瓦片布局的基础，一起看看有趣的东西。

使用 repeat() 时，可以不只按一个轨迹重复，而是按一种轨迹模式重复。

```
.app {
  display: grid;
  grid-column-gap: 15px;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr) 150px);
}

```

这次告诉我们的 Grid 重复尽可能多的集合，其中包括弹性的宽为 **300px** 的列和固定的 **150px** 列。

![](https://gedd.ski/img/minmax-pattern.jpg)

这种布局的工作方式，除了宽为 **minmax** 的弹性列在 **auto-fit** 算法完成后才能得到一些额外的空间外，和之前的布局一样。

## 改变他们的瓦片布局

我之前提过 Twitter 时刻页面只是一个瓦片布局，其中一些网格项跨越更多的单元格。我们一起做些类似的事情，让每个第三个网格跨越两列，第五个网格跨两行。

```
.item:nth-of-type(3n) {
  grid-column: span 2;
}

.item:nth-of-type(5n) {
  grid-row: span 2;
}

```

![](https://gedd.ski/img/tile-variation.jpg)

结果相当酷，但如果我们使用 **dense** [网格项目放置](https://gedd.ski/post/grid-item-placement/)，会看起来更好。

```
.app {
  grid-auto-flow: dense;
}

```

![](https://gedd.ski/img/dense-tiles.jpg)

这会使网格返回到先前跳过的单元格，并尽可能密集地打包列。

## 现在举一个实际例子

谷歌音乐，Twitter和博客布局都很好。但一起看看更具启发性的东西：[Sam](https://gedd.ski/post/solve-it-once/) 的照片集——世界上最好的狗。

我将要添加的是 **grid-auto-rows**，可以使所有 **auto-fit** 的行生成 **200px** 的高。

```
.gallery {
  display: grid;
  grid-gap: 15px;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  grid-auto-rows: 200px;
  grid-auto-flow: dense;
}

.wide {
  grid-column: span 2;
}

.tall {
  grid-row: span 2;
}
```
![](https://gedd.ski/img/sam-gallery.jpg)

Awww 多好的一个小动物，在瓦片布局中看起来更迷人。

## 如果你只知道 Grid 的力量

CSS Grid 很棒，我们才刚开始发现可以用它做什么。我喜欢用它做[杂志布局](https://gedd.ski/post/overlapping-grid-items/)，[游戏网站](https://gedd.ski/post/new-grid-site/)，甚至我正在构建的游戏本身。

我认为瓦片布局是一个用 CSS Grid 解决的问题。这里有一个可以运行的代码[示例](https://codepen.io/geddski/pen/eKyEOM)。

下次你需要一个瓦片布局时，CSS Grid 的 **auto-fit** 和 **minmax** 会让你满意！

<a href="https://gridcritters.com">
  <img src="https://gedd.ski/gc/grid-critters-footer.jpg" />
</a>

掌握 CSS Grid 从玩这个新游戏开始。你将会学习到 Grid 的细节，并从某种破坏中拯救一个可爱的外星生物。

<center>掌握 [CSS GRID](https://gridcritters.com)</center>

> 原文地址：https://gedd.ski/post/tile-layouts/