title:css4选择器：我们可以期待什么
author:李姝楠
date:2017-05-17
categories:
-前端技术
-css
tag:
-css
-css4
-选择器

**important：由于工作在 [Selectors Level 4](https://drafts.csswg.org/selectors/) 提案的生态中，这项提案遵从2017年1月20日出的提案指导，它里面原有的东西将会被改变或消失。**

css已经[转变了20年](https://www.w3.org/Style/CSS20/)，一些人仍然像2002年一样在使用它。我一直觉得css这个强有力的语言经常被抛在项目进度的后面，并被认为是仅仅需要被做，不关心质量、优化、简洁的东西。值得庆幸的是，这种心态随着大家对前端技术兴趣的提升在近几年已经开始改变。

在我看来，一个好的app是不能缺少好的css。网站能被大量访问很大程度上依赖于高质量的css。有时，很难实现一个明确的布局或样式，这导致我们最后写出一个不好的样式，影响用户体验。有一个好消息，css4允许我们用一些好的新选择器，它将简化编写样式的方式，保持样式是简单的、有组织的。

在这篇文章中，我想分享一些正在[w3c](https://www.w3.org/)上讨论的观点。

一起看一下：

## 逻辑组合
### 不包含:not(selector1,selector2)
这个伪类在css3中就已经出现，但只允许一个选择器作为参数。现在，我认为你想添加多少个选择器，它都能很好的支持。它仅用一列选择器作为参数，代表不包含这个参数的元素。

### Example:
```
input:not([required][type="text"]){...}
*:not(.hidden){...}
```

### 匹配所有：matches(selector1,selector2)
这个伪类和:not相反，它用一列选择器作为参数，代表匹配参数的所有元素。
### Example:

```
li:matches(:first-child, .highlighted) { … }
a:matches(*:hover, *:focus) { … }
```


### 相关的：has(selctor1,selector2)
has用一组相关的选择器作为参数，匹配参数中任何一个选择器的元素。
### Example:

```
li:has(> ul) { … }
dt:has(+ dt) { … }
```



## 属性选择器
### case-sensitivity
case-sensitivity的默认行为是依赖于document语言的属性值和名字。用这个新选择器，可以匹配任何具体的属性值相等的元素。我们只需要在邻近的（]）之前添加一个i来表示case-sensitivity。
### Example:

```
/* All input elements with a value of name, Name, NAME, etc, will be matched */
input[value="name" i] { … }
```



## 语言伪类
### 方向性伪类：:dir()
方向性伪类允许我们写基于元素方向的选择器(从左到右，从右到左)，由document语言决定。
### Example:

```
blockquote:dir(ltr) {
  border-left: 5px solid #555;
}
blockquote:dir(rtl) {
  border-right: 5px solid #555;
}
```


### 语言伪类 :lang()
:lang代表一个元素，它是以参数的形式列出的语言。它最先在css2中被引入，但在css4中，我们有可能添加通配符匹配。
### Example:

```
a:lang(es-CR) {
  background-color: blue;
}
/* The following example will match all anchor elements with a language ending in CA like en-CA or fr-CA */
a:lang(*-CA) {
  background-color: red;
}
```



## 位置伪类
### 超链接伪类 :any-link
它是指元素表现的像超链接一样有源锚点(有href属性的元素)。换句话说，任何元素都可以用:link和:visited匹配。就像:matches(:link,:visited)的简写。
### Example:

```
#navbar a:any-link {
  text-decoration: none;
}
```


### 参考元素伪类 :scope
这个选择器在我们想给css规则一个范围时有用。它允许我们制定一套css规则，能为匹配选择器提供一个参考点，像DOM操作中的querySelector()，或html5中加scoped的<style>标签。
### Example:

```
<div>
  <p>Paragraph outside the scope.</p>
  <div>
    <style scoped>
      :scope {
        background-color: black;
      }
      p {
        color: white;
      }
    </style>
    <p>Paragraph is inside the scope.</p>
  </div>
</div>
```


## 时间范围伪类
这类伪类有助于在时间范围中给当前显示的或活跃的位置分类，像一个文件的演讲或用WebVTT为视频展示字幕。

### :current
它代表一个当前正被展示的元素或元素的祖先。并且，它可能提供一系列选择器。它是指:current元素同时匹配参数中的元素，如果没有匹配到，将会去匹配:current元素最近的祖先元素。
### Example:

```
:current(p) {
  background-color: lightgreen;
}
```


### :past
它指任何定义在:current元素之前的元素。
### Example:
```
:past(p) {
  background-color: yellow;
  opacity: .5;
}
```


### :future
它指任何定义在:current元素之后的元素。

```
:future(p) {
  background-color: lightblue;
  opacity: .9;
}
```



## 用户行为伪类
### :focus-ring
匹配一个:focus元素，但同时，UA确定:focus是否指代当前元素(通过'focus ring')。就像点击按钮或链接时一样。
### Example:

```
/* This will match any focused element */
.content > :focus {
  outline-width: 2px;
  outline-style: solid;
  outline-color: lightblue;
}
/* This will match just elements that display a focus ring by default, like buttons or input fields */
.content > :focus-ring {
  outline-width: 2px;
  outline-style: solid;
  outline-color: lightblue;
}
```


### 拖拽伪类:drop和:drop()
:drop指用户正在拖拽且将要被放下的所有元素。:drop()和:drop一样，但允许定义其他过滤器，它可以排除一些drop目标。过滤器可以有：
active: 目标是拖拽交互的当前目标
valid:如果拖放目标对被拖动的项目有效（根据文件语言），则匹配。否则匹配所有拖放目标。例如：这在它们接受文件夹我们想给拖放目标加样式时有用
invalid:匹配当前拖拽目标无效的元素(根据文件语言)，否则什么都不匹配。
参数可设置多个过滤器
### Example:

```
.drop-zone:drop(valid active) {
  background-color: lightgreen;
}
```



## input伪类
### input控制状态

#### 易变性伪类：:read-only和:read-write
:read-write被定义成匹配可由用户改变的元素。否则匹配:read-only。
#### Example:

```
input:read-write {
  border-color: green;
}
input:read-only {
  border-color: gray;
}
```
#### :placeholder-shown
匹配input中有placeholder文本的元素。
#### Example:

```
input:placeholder-shown + label {
  display: none;
}
```

#### :default
这适用于默认在一组相似元素中的一个或多个UI elements。通常适用于菜单，按钮和select列表。例如，会匹配一组按钮中的默认提交按钮。
#### Example:

```
.btn {
  background-color: lightgray;
}
.btn:default {
  background-color: lightgreen;
}
```


### input值的阶段
#### :indeterminate
此伪类应用于具有不确定状态值的UI元素。例如，单选按钮和复选框就属于值不确定，可以选也可以不选。
#### Example:

```
:indeterminate, :indeterminate + label {
  background: yellow;
}
```


### input 值判断
#### :valid和:invaild
一个元素是有效或无效取决于文件语言定义的数据有效性语义。元素没有数据有效性语义时既不是:valid也不是:invalid。
#### Example:

```
<input type="text" required />
```
```
input:invalid {
  border-color: red;
}
```


#### :in-range和:out-of-range
此伪类应用于有范围限制的元素。元素是:in-range或:out-of-range取决于它的值在范围限制（文件语言定义的）之内还是之外。如果一个元素没有数据范围限制或没有形式控制，它既不是:in-range，也不是:out-of-range。
#### Example:

```
<input type="number" required max="5" />
```
```
input:out-of-range {
  border-color: red;
}
```


#### :required和:optional
一个表单元素是:required或:optional，取决于它的值是否可选或必须，另外，必须或可选是相对于表单提交前。非表单元素既不是:required也不是:optional
#### Example:

```
input:optional {
  color: gray;
}
```


#### :user-invalid
它代表有错误值的元素，但只存在于用户交互之后。
#### Example:

```
input:user-invalid {
  border-color: red;
}
```

## 树结构伪类
### :blank
此伪类类似:empty，但另外还匹配只包含被空格影响的元素。
### Example:

```
<p>
</p>
```
```
p:blank {
  display: none;
}
```

## 选择符
### 后代选择器（）或>>
这个选择器描述在dom树中一个元素的后代元素。和分离复合选择器一样有空格是必要的，但这样更明确些。
### Example:

```
h2 >> span { … }
```



## 网格结构选择器

### 列组合 ||
它匹配任何任何在网格或表格中属于列的单元格。
### Example:

```
<table>
  <col>Title</col>
  <col class="selected">Name</col>
  <tr>
    <td>Architect</td>
    <td>John Doe</td>
  </tr>
  <tr>
    <td>Software Developer</td>
    <td>Donkey Kong</td>
  </tr>
</table>
```
```
col.selected || td {
  background: lightblue;
  font-weight: bold;
}
```


### :nth-column()
匹配在网格或表格中属于第几列的单元格。
### Example:

```
:nth-column(2n) {
  background: gray;
}
```


### :nth-last-column()
代表网格或表格中的最后一列。
### Example:

```
:nth-last-column(3n+1) {
  background: purple;
}
```




## 结论
如你所见，有大量新的有力的选择器将会使我们的工作更容易。记住它们会更好！然后，我想澄清这个信息是基于2017年1月20号“Selectors Level 4”的编辑草案,会随时改变。——想要从Arturo了解更多吗？关注我们的博客和[Twitter](https://twitter.com/gorillalogic?lang=en)。