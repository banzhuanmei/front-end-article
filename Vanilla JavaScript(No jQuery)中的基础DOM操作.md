title:css4选择器：Vanilla JavaScript(No jQuery)中的基础DOM操作
author:李姝楠
date:2017-05-17
categories:
-前端技术
-DOM
tag:
-js
-Vanilla JavaScript

>原文链接：https://www.sitepoint.com/dom-manipulation-vanilla-javascript-no-jquery/#modifyingthedom

无论何时，我们需要操作dom时，都会快速想到jquery。然而，vanilla javascript 的dom api实际上已经相当可以了，并且自从ie11以下被[正式放弃](https://www.microsoft.com/en-us/WindowsForBusiness/End-of-IE-support)后，它就可以毫无顾虑地被使用了。

在这篇文章中，我将证明如何用plain js 完成大部分公共的dom操作任务，即：
- 查找并修改dom
- 修改类和属性
- 监听事件
- 动画

我将会展示如何创建你自己的轻量dom库，且可以使用在任何项目中。沿着这种方式，你会知道用vanilla js操作dom并不遥远，事实上许多jquery的方式跟原生的api是相同的。

所以一起探索下…

## dom操作：查找dom
*注意：我不会从细节上解释vanilla dom的api，只涉及表面。在使用的例子中，你可能会看到我没有明确介绍过的方法。这种情况时，请参考非常不错的 [Mozilla Developer Network](https://developer.mozilla.org/en-US/)查看详情。*

dom查找可以用.querySelector()，此方法接受一个任意的css选择器作为参数。

```
const myElement = document.querySelector('#foo > div.bar')
```

它会返回第一个匹配的元素。相反，可以检查一个元素是否匹配选择器：

```
myElement.matches('div.bar') === true
```

如果想得到所有匹配的，可以用：

```
const myElements = document.querySelectorAll('.bar')
```

如果已经有一个可参考的父元素，我们可以只查找这个元素的子元素，而不是所有的dom，有一个像这样缩小的语境，可以简化选择器且提高性能。

```
const myChildElemet = myElement.querySelector('input[type="submit"]')
// Instead of
// document.querySelector('#foo > div.bar input[type="submit"]')
```

那么为什么要去用其他一点都不方便的方式，像.getElementsByTagName()？一个重要的区别是.querySelector()的结果不是动态的，所以当我们动态添加一个元素(看++第三部分++了解详情)去匹配时，结果不会更新。

```
const elements1 = document.querySelectorAll('div')
const elements2 = document.getElementsByTagName('div')
const newElement = document.createElement('div')

document.body.appendChild(newElement)
elements1.length === elements2.length // false
```

另外一个考虑是，这样一种动态收集不需要有所有的信息，而.querySelectorAll()会立即在一个静态表中聚集所有东西，[性能低](https://www.nczonline.net/blog/2010/09/28/why-is-getelementsbytagname-faster-that-queryselectorall/)。

### Working with Nodelists
有两个普遍的关于.querySelectorAll()的陷阱。第一个是我们在结果上不能调用节点方法并将其传播到它的元素上(像你可能用jquery那样)，必须明确地迭代那些元素。另外一个陷阱是：返回值是一个节点列表，不是一个数组。这意味着通常的数组方法直接不可用。有一些对应节点列表的方法，例如.forEach，但它在ie下仍不支持。所以首先必须把节点列表转为数组，或借调数组原型方法。

```
// Using Array.from()
Array.from(myElements).forEach(doSomethingWithEachElement)

// Or prior to ES6
Array.prototype.forEach.call(myElements, doSomethingWithEachElement)

// Shorthand:
[].forEach.call(myElements, doSomethingWithEachElement)
```

每一个元素也有几个引用'family'的自解释、只读属性，这些所有的都是动态的：

```
myElement.children
myElement.firstElementChild
myElement.lastElementChild
myElement.previousElementSibling
myElement.nextElementSibling
```

[元素](https://developer.mozilla.org/en-US/docs/Web/API/element)接口继承于[节点](https://developer.mozilla.org/en-US/docs/Web/API/Node)接口，下面的属性也是可用的：

```
myElement.childNodes
myElement.firstChild
myElement.lastChild
myElement.previousSibling
myElement.nextSibling
myElement.parentNode
myElement.parentElement
```

前面的只参考元素，后面的(除了.parentElement)可以是任何种类的节点，例如文本节点。我们可以检查一个节点的[类型](https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeType)。eg:

```
myElement.firstChild.nodeType === 3 // this would be a text node
```

任何对象，都可以用instanceof检查一个节点的原型链。

```
myElement.firstChild.nodeType instanceof Text
```

## 修改类和属性
修改元素的类和下面一样容易：

```
myElement.classList.add('foo')
myElement.classList.remove('bar')
myElement.classList.toggle('baz')
```

你可以通过读[quick tip by Yaphi Berhanu](https://www.sitepoint.com/add-remove-css-class-vanilla-js/)，更深入地了解如何修改类。元素属性可以像任何对象属性一样被访问。

```
// Get an attribute value
const value = myElement.value

// Set an attribute as an element property
myElement.value = 'foo'

// Set multiple properties using Object.assign()
Object.assign(myElement, {
  value: 'foo',
  id: 'bar'
})

// Remove an attribute
myElement.value = null
```

也有.getAttribute(),.setAttribute()和.removeAttribute()方法，可以直接修改元素的html属性（与提升dom性能相反），这会导致浏览器重绘（你可以用浏览器开发工具检查元素，观察它的变化）。不只是浏览器重绘比设置dom属性消耗多，这种方式也会有[意外的后果](https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute#Notes)。

一般来说，元素没有对应的dom属性时才会去使用这些方法（像colspan），或着如果你真想坚持改变html（例如当克隆一个元素或修改父级的innerhtml，且希望元素不变时——++第三部分++）。

### 添加css样式
css规则可以像其他属性一样被应用，注意在js中属性要写成驼峰形式：

```
myElement.style.marginLeft = '2em'
```

如果想要某些值，可以通过.style属性获得。然而，它将只会给出已经被明确应用的样式。为了得到计算值，可以用.window.getComputedStyle()。它会返回一个 [CSSStyleDeclaration](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration)，包含元素自己的和继承于父级的所有styles：

```
window.getComputedStyle(myElement).getPropertyValue('margin-left')
```

## 修改dom
可以像下面一样移动元素：

```
// Append element1 as the last child of element2
element1.appendChild(element2)

// Insert element2 as child of element 1, right before element3
element1.insertBefore(element2, element3)
```

如果不想移动元素，而是插入一个复制项，可以像下面一样复制：

```
// Create a clone
const myElementClone = myElement.cloneNode()
myParentElement.appendChild(myElementClone)
```

.cloneNode()方法接受一个可选的布尔值作为参数；如果设置为true，将是深度克隆，意味着子元素也会被复制。

当然，也完全可以创建新的元素或文本节点：

```
const myNewElement = document.createElement('div')
const myNewTextNode = document.createTextNode('some text')
```

可以像上面一样插入创建的元素。如果想删除元素，不能直接那么做，但可以从父元素身上删除子元素，像这样：

```
myParentElement.removeChild(myElement)
```

这使我们多了一些工作，实际上通过参考父元素，可以直接删除一个元素：

```
myElement.parentNode.removeChild(myElement)
```

### 元素属性
每一个元素都有.innerHTML和.textContent属性(.innerText和.textContent相似，但有一些[重要的区别](http://perfectionkills.com/the-poor-misunderstood-innerText/))。它使修改html和text有区别。它们是可写入的属性，意味着可以直接修改元素和它们的内容：

```
// Replace the inner HTML
myElement.innerHTML = `
  <div>
    <h2>New content</h2>
    <p>beep boop beep boop</p>
  </div>
`

// Remove all child nodes
myElement.innerHTML = null

// Append to the inner HTML
myElement.innerHTML += `
  <a href="foo.html">continue reading...</a>
  <hr/>
`
```

像上面一样给html添加标记通常不是一个好主意，因为会丢掉元素先前改变的属性和事件监听(除非像在++第二部分++中那样修改html属性)。设置.innerHTML对完全丢掉标记、用其他东西替代它是比较好的，例如：服务端渲染标记。所以append元素会比下面这种好些：

```
const link = document.createElement('a')
const text = document.createTextNode('continue reading...')
const hr = document.createElement('hr')

link.href = 'foo.html'
link.appendChild(text)
myElement.appendChild(link)
myElement.appendChild(hr)
```

然而用这种方式，会造成两次浏览器重绘，一次是append元素，一次是修改了innerHTML的地方。针对这个问题，可以先把所有的节点集合到[DocumentFragment](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment)，只append单个的fragment：

```
const fragment = document.createDocumentFragment()

fragment.appendChild(text)
fragment.appendChild(hr)
myElement.appendChild(fragment)
```

## 监听事件
这可能是最好的绑定事件方式：

```
myElement.onclick = function onclick (event) {
  console.log(event.type + ' got fired')
}
```

但一般都不用这种方式。.onClick是元素的属性，意味着你可以修改它，但是不能给它加额外的监听，通过重新指向一个新的函数可以重写.onClick。

另外，可以用更强大的.addEventListener()添加想要的各种事件类型。它需要三个参数：事件类型(像click)、一个无论何时出现在元素身上的事件的响应函数(可以传递event对象)和一个可选的配置对象，用来说明更深操作。

```
myElement.addEventListener('click', function (event) {
  console.log(event.type + ' got fired')
})

myElement.addEventListener('click', function (event) {
  console.log(event.type + ' got fired again')
})
```

在监听函数中，event.target代替正被事件触发的元素(除非使用[箭头函数](https://www.sitepoint.com/es6-arrow-functions-new-fat-concise-syntax-javascript/))。因此可以很容易地取到它的属性，像这样：

```
// The `forms` property of the document is an array holding
// references to all forms
const myForm = document.forms[0]
const myInputElements = myForm.querySelectorAll('input')

Array.from(myInputElements).forEach(el => {
  el.addEventListener('change', function (event) {
    console.log(event.target.value)
  })
})
```

### 阻止默认行为
在事件监听中，event总是可用的，这对需要时都可以传递过来是非常好的体验(当然可以给它起一个任何你想要的名字)。不用[Event](https://developer.mozilla.org/en-US/docs/Web/API/Event)接口阐明，.preventDefault()是一种更好的阻止浏览器默认行为的方式，例如一个链接。另一个普遍的案例是客户端form提交失败时，有条件地阻止form提交。

```
myForm.addEventListener('submit', function (event) {
  const name = this.querySelector('#name')

  if (name.value === 'Donald Duck') {
    alert('You gotta be kidding!')
    event.preventDefault()
  }
})
```

另一个重要的事件是.stopPropagation()，它会阻止元素的冒泡事件。这意味着在一个元素身上有阻止点击事件传播时，另一个点击事件在它的父级身上，子元素身上的点击事件不会触发父级，否则，将会都触发。

.addEventListener() 的第三个参数是可配置的，它可以是以下任何一个boolean属性(默认都为false):
- capture:事件将在子元素之前被触发(事件捕获和事件冒泡有自己的文章，了解详情请戳[这里](http://javascript.info/tutorial/bubbling-and-capturing))
- once:正如你猜测的，指事件是否只被触发一次
- passive:指 event.preventDefault() 将会被忽略(通常会在console里面加警告)

最常用的参数是.capture ;实际上，有一个很常见的简写：替代在配置对象中定义它，你可以传一个boolean值：

```
myElement.addEventListener(type, listener, true)
```

事件监听可以用.removeEventListener()取消掉，它会把指定的事件类型和回调函数删掉；比如，once参数的功能也可以这样写：

```
myElement.addEventListener('change', function listener (event) {
  console.log(event.type + ' got triggered on ' + this)
  this.removeEventListener('change', listener)
})
```

### 事件代理
另一个有用的模式是事件代理：想给form中的所有input子元素添加一个change事件监听。一种是用myForm.querySelectorAll('input')给所有元素加.addEventListener。然而这是不必要的，也可以加到form身上，检查event.target的内容。

```
myForm.addEventListener('change', function (event) {
  const target = event.target
  if (target.matches('input')) {
    console.log(target.value)
  }
})
```

这种模式的另一种优势是会自动给动态添加的子元素加事件，不需要绑定新的监听。

## 动画
通常，添加动画最简单的方式是用加了transition属性的css类，或者用@keyframes。但如果你需要更灵活的(例如游戏)，可以用JavaScript操作。

比较稚拙的实现动画方式是使用定时器。但会导致文档多次重排，效率很低，而且很卡，尤其在移动设备上。相反，我们可以用window.requestAnimationFrame来同步更新，它会给浏览器重绘排好所有当前的变化。这种方式接收一个回调函数作为参数，指当前时间戳：

```
const start = window.performance.now()
const duration = 2000

window.requestAnimationFrame(function fadeIn (now)) {
  const progress = now - start
  myElement.style.opacity = progress / duration

  if (progress < duration) {
    window.requestAnimationFrame(fadeIn)
  }
}
```

用这种方式可以实现流畅的动画。想要了解更多，戳Mark Brown的[这篇](https://www.sitepoint.com/quick-tip-game-loop-in-javascript/)文章。

## 编写自己的辅助方法
跟jQuery简洁的链式操作 $('.foo').css({color: 'red'})比起来，总去遍历元素确实很麻烦。所以为什么不写一些想要的简单的方法？

```
const $ = function $ (selector, context = document) {
const elements = Array.from(context.querySelectorAll(selector))

  return {
    elements,

    html (newHtml) {
      this.elements.forEach(element => {
        element.innerHTML = newHtml
      })

      return this
    },

    css (newCss) {
      this.elements.forEach(element => {
        Object.assign(element.style, newCss)
      })

      return this
    },

    on (event, handler, options) {
      this.elements.forEach(element => {
        element.addEventListener(event, handler, options)
      })

      return this
    }

    // etc.
  }
}
```

因此，我们有一个非常小的DOM库，包含想要的方法，并且不用考虑向后的兼容性。通常，有一些原型中的方法。这里有实现辅助函数的[方法](https://gist.github.com/SitePointEditors/4f6643d62a87aece860b0784c6eeffd2)。另外，辅助函数应该像下面一样简洁：

```
const $ = (selector, context = document) => context.querySelector(selector)
const $$ = (selector, context = document) => context.querySelectorAll(selector)

const html = (nodeList, newHtml) => {
  Array.from(nodeList).forEach(element => {
    element.innerHTML = newHtml
  })
}

// And so on...
```
## 例子
为了更好的总结这边文章，这个CodePen上演示了许多以上实现简单技术的概念。我不推荐你花时间浏览源码，如果你有任何留意或问题，请在下面的评论中让我知道。

<iframe id="cp_embed_aJaggB" src="https://codepen.io/SitePoint/embed/aJaggB?height=360&amp;theme-id=6441&amp;slug-hash=aJaggB&amp;default-tab=result&amp;user=SitePoint&amp;embed-version=2&amp;pen-title=Tightbox" scrolling="no" frameborder="0" height="360" allowtransparency="true" allowfullscreen="true" name="CodePen Embed" title="Tightbox" class="cp_embed_iframe " style="width: 100%; overflow: hidden;"></iframe>

## 结论
我希望可以用简单的javaScript来说明DOM操作，事实上，许多jQuery方法等价于原生DOM API。这意味着对于每天常用的一些东西，额外的DOM库开销是不必要的。

而且，部分原生API是冗余或不方便的(例如必须一直手动遍历节点列表)，我们可以很容易地写一个简单的辅助函数，摘出重复的任务。

现在结束了，你有什么想法？你会更忽略第三方库，或消耗不必要的开销？请在下方的评论区让我知道。
