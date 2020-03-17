有了Flexbox,一种新的布局模式在CSS3中出现了，从表面理解，就是可以把很多卡片放在一行。你可能注意到，卡片布局已经比过去几年受欢迎，网站都包含它。Pinterest 和 Dribbble 使用卡片使信息和视觉更具特征。如果你进入Material Design，谷歌的卡片在他们的模式库中有很好的描述。

我个人喜欢卡片布局是因为他的可读性和可滚动。它用一种易浏览、滚动、立马看到所有东西的方式，完美的凸显了信息。

## 如何创建一个卡片布局
如果你尝试过一些偶数高度的内容，肯定知道很不容易。过去可能需要费很多功夫，有了Flexbox，我保证那些日子会成为过去。取决于你提供的浏览器支持水平，可能会需要包含一些回退，但浏览器支持。安全起见，请务必查看 Can I use。

[And remember, you should never make changes on your live site. Try experimenting with Local by Flywheel instead, a free local WordPress development app. Download it today!](https://local.getflywheel.com/)

Flexbox的原理是给容器的display设为`flex`,
会“flex”所有容器。同高的列、缩放、和配置可以简化创造一个好的布局。卡片是更多是对Flexbox的基础介绍，一旦掌握基础，可以做出更灵活的布局。

## Flexbox 和 versatility
卡片是多功能且吸引人的，容易在大小设备上交互，适用于响应式设计。每一个卡片作为一个容器很容易缩放。随着屏幕变小，每一行的卡片数量减少，并垂直放置。还能固定或改变高度。

## 如何创建一个布局
创建一个flex卡片布局，大屏幕上一行可以放四列，中屏上放两列，小屏幕上只能放一列。
![image](https://getflywheel.com/wp-content/uploads/2017/02/use-flexbox-card-design-device-comparison.png)

下面是创建了一个展示卡片的基础布局的代码片段。没包含卡片内容(因为它在代码示例中太长了)，一定要在开始时放入一些内容(在四个卡片放入不同数量的内容)。另外，这里展示的是一行四个卡片，但如果你想放更多行，可以在 [Codepen](http://codepen.io/collection/XLRrqW/) 上查看代码。

为了用网格展示卡片，需要从外部开始，确定参照物是正确的很重要，否则会有些混乱。

带`.cards`类的的 section 是第一个目标。它的 display 属性需要改为 `flex`。

下面是开始的代码：


```
<div class="centered">
 
            <section class="cards">
                 
                <article class="card">
                   <p>content for card one</p>
                </article><!-- /card-one -->
 
    <article class="card">
                   <p>content for card two</p>
            </article><!-- /card-two -->
 
<article class="card">
                  <p>content for card three</p>
            </article><!-- /card-three -->
 
<article class="card">
                   <p>content for card four</p>
            </article><!-- /card-four -->
 
    </section>
</div>
```

CSS:


```
.cards {
   display: flex;
   justify-content: space-between;
}
```

### Flex 属性

在深入了解之前，最好先了解 flex 属性。flex 属性指定子项目相对容器剩余项目的长度。flex 是 `flex-grow`, `flex-shrink` 和 `flex-basis` 的简写。默认值是 `0 1 auto`；我认为理解 Flexbox最好的方式是设置不同的值看下发生了什么。

`flex-grow` 定义 flex 子项目占容器的大小。

![image](https://getflywheel.com/wp-content/uploads/2017/02/use-flexbox-card-design-flex-grow-diagram.jpg)

`flex-shrink` 定义相对容器中的其他子项目如何收缩。

![image](https://getflywheel.com/wp-content/uploads/2017/02/use-flexbox-card-design-flex-shrink.jpg)

`flex-basis` 指定子项目最初的大小，定义的是 content-box 的大小，除非使用 box-sizing 指定。当宽度由内容定义时，Auto 是默认值，类似于 width：auto，将占据它内容大小的空间。也可以设置指定值比如：`flex-basis:15rem`。如果设为0，表现是正常的，因为子项目不会扩展占据剩余空间。

![image](https://getflywheel.com/wp-content/uploads/2017/02/use-flexbox-card-design-flex-basics.jpg)

先加上 `display:flex;` 和 `justify-content:space-between;`,在这一点上，有些不可预测。尽管现在不明显，Flexbox 还是将会被使用。有了这个声明，每一个子项目会被一个接一个放在一横排。

![image](https://getflywheel.com/wp-content/uploads/2017/02/step-01-flexbox-uneven-boxes.jpg)

在 [Codepen](https://codepen.io/abbeyjfitzgerald/pen/YNLgjQ) 上查看。

你可能很好奇为什么每一个子项目有不同的宽。Flexbox 试图找出每个子项目的最小默认宽度。由于各种字长和其他设计元素，你最终会得到这些不同大小的盒子。为了获得一致的外观，我们需要做更多的工作。
设置 wrap 并确定所需的宽度将有助于将它们制成统一的卡片。


```
.cards {
   display: flex;
   flex-wrap: wrap;
   justify-content: space-between;
}
```

默认情况下，flex项目都将尝试放在一行。添加 `flex-wrap:wrap;` 可以使项目一个在一个下面，因为默认全宽。

![image](https://getflywheel.com/wp-content/uploads/2017/02/step-02-full-width-boxes.jpg)

在 [Codepen](http://codepen.io/abbeyjfitzgerald/pen/EZLzer) 上查看。

全宽适用于小设备，当我们在加不同的断点前放大屏幕时记住这一点。改变宽度时，卡片看起来更均匀。

需要加 `.card` 类给卡片加样式。


```
.cards {
   display: flex;
   flex-wrap: wrap;
   justify-content: space-between;
}
 
.card {
    flex: 0 1 24%;
}
```

记住之前的，flex 属性是 `flex-grow:0` `flex-shrink:1` `width:24%` 的简写。通过添加特定宽，可以得到一排包含空隙的四个卡片。

![image](https://getflywheel.com/wp-content/uploads/2017/02/step-03-evenly-spaced-boxes.jpg)

在 [Codepen](http://codepen.io/abbeyjfitzgerald/pen/vgjwVN) 上查看。

可以为间距设置 `justify-content` 属性。第一个强制显示在左边，第二、第三显示在中间，第四个强制显示在最右边。因为卡片宽为24%，四个不到100%，所以有一些空间，还剩4%被平均分配，所以在卡片之间有1.33%的空间。

![image](https://getflywheel.com/wp-content/uploads/2017/02/card-height-comparison.jpg)

在 [Codepen](http://codepen.io/abbeyjfitzgerald/pen/QdxEym) 上查看。

使用 `calc` 可以更精确。修改 `flex-basis` 如下：

```
.card {
    flex: 0 1 calc(25% - 1em);
}
```
卡片会占据25%减掉 1em 的空间，稍微小一点。

这是一种调整可用空间的方式。
1em 在子项目之间均匀分布，最终得到完美的布局。

至今我们还没谈过高度。我添加了另一行来演示高如何起作用。取决于哪个卡片的内容最多，其他的会跟它一样高。因此每行会有相同的高度。

这是一个放大的视图，你会看到第一行高度一样，因为其中的第二个卡片的内容比较多。第二行内容少，所以整体高度小。

### 用于更小设备的卡片

目前我们屏幕有四列，
这不是很好的体验。如果缩小浏览器窗口，会看到四个卡片被压扁了，这对可读性很不理想。幸好有媒体查询，有了它会看起来更好。

![image](https://getflywheel.com/wp-content/uploads/2017/02/cards-squished-in-size.jpg)

为了解决问题，要制定断点确保在所有不同屏幕上可以很好的展示。

下面这些断点将会被用到：


```
@media screen and (min-width: 40em) {
    .cards {
   }
 
    .card {
    }
}
 
@media screen and (min-width: 60em) {
    .cards {
   }
     
    .card {
     }
}
 
 
@media screen and (min-width: 52em) {
    .centered {
        
    }
}
```

直到现在，这是一个大框架，让我们从最小宽为 `min-width:40em`的断点开始。


```
@media screen and (min-width: 40em) {
    .cards {
        display: flex;
        flex-wrap: wrap;
        justify-content: space-between;
    }
 
    .card {
        flex: 0 1 calc(25% - 1em);
    }
}
```

有了这些变化，卡片在屏幕小于 640px 时一个在一个下面布满整个屏幕。如果浏览器窗口大于这个值，会得到四列。这样是对的，因为 `min-width` 为 40em，这就是创建四列的分界点。

这里少了处于中间大小的屏幕。对于中间屏幕，一行两个卡片可读性比较好。在添加一行两个卡片之前，需要先添加一个兼容大屏幕的处理，它一行可以放四个卡片。

```
@media screen and (min-width: 40em) {
    .cards {
        display: flex;
        flex-wrap: wrap;
        justify-content: space-between;
    }
 
    .card {
        flex: 0 1 calc(50% - 1em);
    }
}
```

有了这个改变，一切可以正常了！缩放浏览器可以保证内容看起来很正常。

![image](https://getflywheel.com/wp-content/uploads/2017/02/breakpoint-examples.jpg)

在 [Codepen](http://codepen.io/abbeyjfitzgerald/pen/wgXWpm) 上查看。

如果你的行看起来很正常，那很好。如果你最后一行不均匀，接着往下读。

## 动态内容和最后一行卡片间距

根据卡片数量，你可能会得到一个看起来不太好的最后一行。如果最后一行是满的或还有一个卡片，这都不是问题。有时你会提前计划好内容，但有时内容是动态的，最后一行可能不会跟你预期的一样。如果最后一行有多张额外的卡片并设置了内容，空间会均匀分配在它们之间，可能会与上面的行表现不一样。

![image](https://getflywheel.com/wp-content/uploads/2017/02/flexbox-last-row-issue.jpg)

为了得到这样的结果，需要不同的思考方式。我觉得这不是很有效，但相对简单。

给 `.cards` 和 `.card` 加媒体查询：


```
.cards {
    display: flex;
    flex-wrap: wrap;
 }
 
.card {
    flex: 1 0 500px;
    box-sizing: border-box;
    margin: 1rem .25em;
}
 
The media queries are where the number of cards is determined:
 
@media screen and (min-width: 40em) {
    .card {
       max-width: calc(50% -  1em);
    }
}
 
@media screen and (min-width: 60em) {
    .card {
        max-width: calc(25% - 1em);
    }
}
```

![image](https://getflywheel.com/wp-content/uploads/2017/02/flexbox-resolved-last-row.jpg)

可以在 [Codepen](http://codepen.io/abbeyjfitzgerald/pen/LWXWYb) 上查看修改了的解决方案。

希望这能给你一个 Flexbox 的基本概述。 浏览器对 Flexbox 有良好的支持，卡片布局将继续在网站设计中被应用。记住卡片布局只是应用 Flexbox 的开始。




