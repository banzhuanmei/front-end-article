![image](https://process.filestackapi.com/cache=expiry:max/resize=width:700/compress/eVQWNOZuTeWJ0cnHqGLN)

照片来源于 [*wu yi*](https://unsplash.com/photos/mk7zhx5lFbc?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上传至 [*Unsplash*](https://unsplash.com/search/photos/elegant%20coffee?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 中。

在语言被发明后不久，我写了第一行 JavaScript 代码。如果你在那个时候告诉我有一天我会写[一系列关于 JavaScript 的优雅模式的文章](https://medium.com/search?q=Elegant%20patterns%20in%20JavaScript)，我肯定会笑。我认为 JavaScript 并不是真正的编程语言。

自那以后的20年变化很大。现在我在 JavaScript 中，看到  Douglas Crockford 写 JS 的好处部分：一个杰出的动态编程语言……具有强大的表达力。

所以，毫不犹豫，这儿有我最近使用在代码中比较好的模式。希望你可以和我一样喜欢。

> 注意：我保证不是我创造了它，而是在别人的代码看到，最终使用在我的代码中。

## 接收一个对象，返回一个对象(RORO)

我现在大部分函数都接收一个单一的`object`，并且许多都返回或解析成一个`object`。
部分归功于 ES2015 引入的解构特性，它是很有用的模式。我给它起了一个很傻的名字：“RORO”。

> 注：解构是现代 JavaScript 中我最喜欢的特性。我们将在本文中充分利用它，如果你不熟悉，可以通过下面这个视频加速了解。

<iframe class="embedly-embed" src="https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fwww.youtube.com%2Fembed%2F-vR3a11Wzt0%3Ffeature%3Doembed&amp;url=http%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3D-vR3a11Wzt0&amp;image=https%3A%2F%2Fi.ytimg.com%2Fvi%2F-vR3a11Wzt0%2Fhqdefault.jpg&amp;args=showinfo%3D0&amp;key=001db74e857f4a33afe674a632f4e8f5&amp;type=text%2Fhtml&amp;card=1&amp;schema=youtube" width="854" height="480" scrolling="no" frameborder="0" allowfullscreen=""></iframe>

下面是几个 RORO 被喜欢的理由：
- 命名参数
- 简洁的默认参数
- 多样的返回结果
- 函数组成简单

一一看下。

## 命名参数

假设有一个函数返回给定角色中的用户列表，并且假设需要两个 option，一个包含每个用户的联系信息，另一个包含非活动用户，一般会写：

```
function findUsersByRole (
  role, 
  withContactInfo, 
  includeInactive
) {...}
```
可能会这么调用：

```
findUsersByRole(
  'admin', 
  true, 
  true
)
```
后面两个参数非常含糊，“true,true” 代表什么？

如果我们的 app 不需要联系信息，但总需要非活动用户会怎么样？我们必须始终添加中间的参数，即使它并不相关(后面会介绍更多)。

总之，这种传统的方法留下了可能含糊不清的代码，难以理解和编写。

用一个单一的 object 作为参数：

```
function findUsersByRole ({
  role,
  withContactInfo, 
  includeInactive
}) {...}
```
函数看起来几乎没变，只是在参数上加了大括号。这表明函数并不是接收三个参数，而是期望一个单一的对象，属性名包括`role` `withContactInfo` `includeInactive`。

这样可以生效的原因是 ES2015 中的解构。

现在可以这样调用：

```
findUsersByRole({
  role: 'admin', 
  withContactInfo: true, 
  includeInactive: true
})
```
这已经不含糊并且很易读理解了。此外省掉或重新排序参数不再是问题，因为它们已经是对象的命名属性了。

例如这样是可以的：

```
findUsersByRole({
  withContactInfo: true,
  role: 'admin', 
  includeInactive: true
})
```
这样也是：

```
findUsersByRole({
  role: 'admin', 
  includeInactive: true
})
```
也可以在不破坏旧代码的情况下加新参数。

这里需要注意的一点是，如果希望所有参数都是可选的，换句话说，如果以下是有效的调用…

```
findUsersByRole()
```
需要给参数设置一个默认值，像这样：

```
function findUsersByRole ({
  role,
  withContactInfo, 
  includeInactive
} = {}) {...}
```
对参数使用解构提升了不变性。解构函数中的对象时，将会把对象的属性赋值给新变量。改变变量的值不会影响原始对象。

像下面的：

```
const options = {
  role: 'Admin',
  includeInactive: true
}

findUsersByRole(options)

function findUsersByRole ({
  role,
  withContactInfo, 
  includeInactive
} = {}) {
  role = role.toLowerCase()
  console.log(role) // 'admin'
  ...
}

console.log(options.role) // 'Admin'
```
即使改变了`role`的值，`options.role`仍然不变。

> 解构是浅拷贝，如果参数的属性值是一个复杂类型(例如数组或对象)，改变它们会影响原始值。

## 清除默认参数
使用 ES2015 可以定义默认参数。实际上，在上面的`findUsersByRole`中，加`= {}`已经是使用了默认参数。

使用传统的默认参数，`findUsersByRole`看起来会是这样：

```
function findUsersByRole (
  role, 
  withContactInfo = true, 
  includeInactive
) {...}
```
如果想把`includeInactive`设置为 `true`,必须给`withContactInfo`设置`undefined`：

```
findUsersByRole(
  'Admin', 
  undefined, 
  true
)
```
多可怕？

对比使用一个对象：

```
function findUsersByRole ({
  role,
  withContactInfo = true, 
  includeInactive
} = {}) {...}
```
现在可以写成：

```
findUsersByRole({
  role: ‘Admin’,
  includeInactive: true
})
```
…并且保留了`withContactInfo`的默认值。

## 所需的参数
你多久写一次这样的东西？

```
function findUsersByRole ({
  role, 
  withContactInfo, 
  includeInactive
} = {}) {
  if (role == null) {  
    throw Error(...)
  }
  ...
}
```
> 注：上面使用`==`测试`null`和`undifined`。

如果我告诉你可以用默认参数来验证所需参数。

首先给`requiredParam()`定义一个抛异常的函数。

像这样：

```
function requiredParam (param) {
  const requiredParamError = new Error(
   `Required parameter, "${param}" is missing.`
  )

// preserve original stack trace
  if (typeof Error.captureStackTrace === ‘function’) {
    Error.captureStackTrace(
      requiredParamError, 
      requiredParam
    )
  }

throw requiredParamError
}
```
> 我知道 *requiredParam* 不是 RORO。这就是为什么我说我的许多函数——并非所有。

现在，可以给`requiredParam`设置`role`默认值，像这样：

```
function findUsersByRole ({
  role = requiredParam('role'),
  withContactInfo, 
  includeInactive
} = {}) {...}
```
上面的代码，如果调用`findUsersByRole`没传`role`，会报`Required parameter,"role" is missing`。

从技术上讲，也可以使用这种技术以及常规的默认参数;
不一定需要一个对象。但这个技巧太不用提了。

## 多样的返回结果

JavaScript 函数只能返回一个单一值，如果这个值是一个`object`，就可以包含更多信息。

函数要把`User`保存到数据库的功能，
当函数返回一个对象时，它可以向调用者提供很多信息。

例如，在保存函数中常见的模式是“更新”或“合并”数据。这意味着在数据库表中插入行(不存在时)或更新数据(已经存在)。

在这个例子中，知道保存函数执行的是`INSERT`或`UPDATE`是很容易的。能精准地显示存入数据库的内容、知道操作状态是比较好的；函数是否执行成功、是否正在等待更大的数据处理、是否超时？

当返回一个对象时，很容易立马传达这些信息。

例如：

```
async saveUser({
  upsert = true,
  transaction,
  ...userInfo
}) {
  // save to the DB
  return {
    operation, // e.g 'INSERT'
    status, // e.g. 'Success'
    saved: userInfo
  }
}
```
技术上，上面返回一个`Promise`，解析为`object`，但你明白就可以。

## 函数组合简单

> “函数组合是合并两个或更多函数产生一个新函数的过程。函数组合就像管道合并，以供数据流传输”——[Eric Elliott](https://medium.com/@_ericelliott)

可以用`pipe`函数组合函数：

```
function pipe(...fns) { 
  return param => fns.reduce(
    (result, fn) => fn(result), 
    param
  )
}
```
上面的函数接收多个函数，返回一个函数从左到右执行这个函数列表，从给定的参数开始，把一个函数的结果传递给下一个函数。

如果你有些困惑，别担心，下面这个例子可以解释清楚。

有一个限制就是列表中的每个函数都只能接收一个单一的参数，但对 RORO 来说，这不是问题！

有一个`saveUser`函数，接收`userInfo`依次按顺序执行 vilidate、normalize、persist三个分离的函数。

```
function saveUser(userInfo) {
  return pipe(
    validate,
    normalize,
    persist
  )(userInfo)
}
```
可以在`validate`，`normalize`和`persist`中使用 rest 参数来解构每个函数需要的值，并将所有的东西都传回给调用者。

这些代码能提供要点：

```
function validate(
  id,
  firstName,
  lastName,
  email = requiredParam(),
  username = requiredParam(),
  pass = requiredParam(),
  address,
  ...rest
) {
  // do some validation
  return {
    id,
    firstName,
    lastName,
    email,
    username,
    pass,
    address,
    ...rest
  }
}

function normalize(
  email,
  username,
  ...rest
) {
  // do some normalizing
  return {
    email,
    username,
    ...rest
  }
}

async function persist({
  upsert = true,
  ...info
}) {
  // save userInfo to the DB
  return {
    operation,
    status,
    saved: info
  }
}
```
## 要不要选择 RORO 是个问题

一开始我说过，我大部分函数会接收一个对象，许多函数也会返回一个对象。

跟其他模式一样，RORO 只是一个工具，在需要通过参数列表更加灵活、返回值更语义清晰的地方使用它。

如果你正在写的函数只需要接收一个单个的参数，则不必要接收一个`object`。另外如果你的函数返回一个值就可以清晰传达信息，也不必要返回一个`object`。

假设如果有一个`isPositiveinteger`函数，判断参数是否是整型，那它一点都不会从 RORO 中收益。























