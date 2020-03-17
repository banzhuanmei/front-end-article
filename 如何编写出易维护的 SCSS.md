![](https://cdn-images-1.medium.com/max/2000/0*QYboKy3YQAlLU1zf)

<center> 由 Ilya Pavlov ​​发布在 Unsplash 上的“电脑屏幕上代码行的特写” </center>

# 编写可维护的 SCSS

本文将提供一些编写样式的提示和技巧，避免您在后期因为繁杂的样式而抓狂。

其中一些技巧是多年来我在网上找到并实际应用过的，发现它们跟宣传的一样好用(很高兴在搜索问题时能遇到相关链接)。还有一些是我在开发过程中注意到的，所有这些技巧都带主观意见，可能还有其他方法可以解决这里提出的一些问题：请在评论中告诉我们！

#### 编写样式时，避免超过三层的深层嵌套(开始)

深层嵌套会输出长选择器字符串，会导致性能下降。

嵌套选择器的权重和优先级都高，因此长选择器会迫使后续选择器需要更大的权重去覆盖样式。

长选择器还降低了样式的可移植性和可维护性。

**当选择器变得很长时，你可能会写出如下 CSS:**

* 强耦合的 HTML(比较脆弱)
* 选择器过于具体(权重较大)
* 无法重复使用
* 如果你写的 HTML 和 CSS 格式良好，就不需要写很多嵌套。

**避免在 React 项目中深层嵌套**

* React 中，通过导入组件级别的样式来模块化 CSS 是很常见的。这对于防止意外的样式流失和明确组件间共享样式提出了固有的挑战。
* 为了防止意外的样式流出，组件根元素应该有唯一的驼峰名称(如：`myCustomModalComponent `)。**永远不要复制根元素的类名**。
* SCSS 文件中第一行应该是根元素的类名，这样所有的样式都限定在当前组件中。
* 根元素下面的元素应遵循 BEM 命名，一般情况下，块内的块可能属于它们自己的 React 组件。然而，块内的块应该开始一个新的“块”，**而不是从现有的块扩展类名**。
* 遵循这种做法，始终可以轻松地把样式移动到共享模块中，因为随着项目的进展，复用需求会变得更加明显。
* 这种做法使样式易于阅读和查找，因为没有深度嵌套。

**识别并编写重复模式(DRY)**

* 编写样式前，确定不同的模块。有的模块在开始时并不明显，当变得明显时，返回把它们当成模块书写
* 重复的值用变量表示
* 利用选择器占位符 %placeholder 编写重复样式
* 使用 mixins，采用变量或重复多行的模式

#### 根据网站中的功能命名变量，而不是按外观命名。
* 例如，用 **`$primary-color`** 命名主题颜色，而不用 **`$red-color`**。

**将 @extend 用于固有的和主题相关的元素。**

**将 mixin 用于共享相同样式的元素。**

* 命名变量时针对功能而不针对样式表现。例如网站上有一个联系人框和参考框，并且它们都是蓝色边框，但我们希望后期可以灵活地更改其他东西，我们可能倾向于把联系人框和参考框跟蓝色框一样，作为一种分类。这不仅会使 CSS 变得混乱，而且很受限制，产生技术债，因为这些类都是后来的开发选择加样式的。这是使用 `@extend` 的一个好场景，因为它们都是蓝色边框，通过变量表示可以使 CSS 更简洁，并可以从 CSS 结构中提取出蓝色边框。

```
%blue-box {
  background: #bac3d6;
  bordeR: 1px solid #3f2adf;
}
.contact-box {
  @extend %blue-box;
  ...
}
.references-box {
  @extend %blue0box;
  ...
}
Generates CSS:
.contact-box, .references-box {
  background: #bac3d6;
  border: 1px solid #3f2adf;
}
```

**不要照搬 DOM 结构**

* 照搬 DOM 结构会使你的 SCSS 几乎完全无法重复使用。
* 它还会在不破坏样式的情况下，使更改 HTML 结构变的非常困难。

**不要过度使用 ID 选择器**

* 因为对 DOM 的更改将使你的样式无效并不可重用，所以使用 ID 选择器比如 #5 就会有这种问题。

**命名空间**

* 命名空间很棒！但它应该定义在组件级别——而不是页面级别。
* 而且，命名空间应按描述性功能级别定义，而不是按页面位置定义。例如，`.profile-header` 可以写成 `.header-hero-unit`。
* 错误的：

```
.nav,.home-nav,.profile-nav,

```
* 正确的：

```
.nav,.nav-bar,.nav-list

```

**样式范围**

页面级别的样式覆盖应该是最小的，并且在单页面级别的嵌套下。

```
.home-page {
   .nav {
      margin-top: 10px;
   }
}

```

**译者注：以下内容为 SUIT CSS 的命名规则，之前没接触过，大家有兴趣可访问 [https://github.com/suitcss/suit/blob/master/doc/naming-conventions.md](https://github.com/suitcss/suit/blob/master/doc/naming-conventions.md)，其中有举例说明，易于理解。**

**命名**

* 元素依赖 javascript 操作时，选择器使用以`.js-`为前缀的类名。
* `.u-`前缀的类名，用于单一用途的实用类，像 .u-underline，u-capitalize等。
* 有意义的连字符和驼峰命名——强调组件、后代组件和修饰符之间的分割。

组件名称必须以驼峰式命名。

`.myComponent { /* … */ }`

`<article class="myComponent"></article>`

* 组件名称——修饰符名称

组件修饰符是以某种形式修改基本组件的类。修饰符名称必须以驼峰式编写，并通过两个连字符与组件名称分隔。除了基本组件类之外，该类也应被包含在 HTML 中。

* 组件名称——后代名称

组件后代是用“-”连接基本组件类和特定后代名组成的后代节点的类。它负责代表特定组件直接向后代节点应用样式。后代命名必须以驼峰式命名。译者注：例如 `Tweet` 中的 `Tweet-header`、`Tweet-bodyText`。

```
<article class="Tweet">
  <header class="Tweet-header">
    <img class="Tweet-avatar" src="{{src}}" alt="{{alt}}">
    …
  </header>
  <div class="Tweet-bodyText">
    …
  </div>
</article>
```

* 带`.is` 前缀的类指有状态的类(通常用 js 切换)，像 .is-disabled。

使用 `is-stateName` 对组件状态进行修改。状态类名必须是驼峰式。永远不要直接给这些类设置样式，它们应该总是和组件基本类在一起被设置(译者注：例如给 `.is-expanded` 设置样式时，应写成 `.Tweet.is-expanded { /* … */ }`)。

JS 可以添加/删除这些类。这意味着可以在多个组件内用相同的状态类名，但每个组件必须定义它自己状态类的样式(因为状态类作用在自己的组件范围内)。

* 有意义的变量名

使用与变量的目的或输出有关的名称，而不是按描述变量值或属性本身命名。

即：如果问题中的变量代表所有悬停状态文字颜色的变化，用`$hover-color` 代替 `$colorName`；用 `$element-name-transition-timing` 代替 `$transition` 设置一个特定元素 `transition` 属性的时间值；`$success-color` 代替 `$green` 等。

> 原文链接：[https://medium.com/@Mike_Flores23/writing-maintainable-scss-cbfa844eb2d0](https://medium.com/@Mike_Flores23/writing-maintainable-scss-cbfa844eb2d0)