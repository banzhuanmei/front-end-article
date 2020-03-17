![image](https://cdn-images-1.medium.com/max/2000/1*EHsl1WfbGv22dCeVCOYufQ.jpeg)

# Chiccocoin：通过在NodeJS中创建区块链来了解什么是区块链


> 拒绝：chiccocoin不是真正的加密货币，我们不会出售它，本文仅用于娱乐/教育目的。

每天在我们吸收的信息中，都会找到关于新加密货币的消息，或者有人说它们是一个即将爆发的大泡沫，而且只有区块链会保留下来。
但是，区块链是什么？

根据定义：

是不断增长的记录列表，称为块，使用加密技术进行链接和保护。

所以，区块链是一个不可变的，顺序的记录链，称为块。
每个块可以包含交易，文件或任何你喜欢的数据。
重要的是他们使用哈希链接在一起。

区块链通过设计是安全的，并且是具有高[拜占庭容错性的分布式](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance)计算系统的示例。这使得区块链可能适用于记录事件，医疗记录和其他记录管理活动，如身份管理，交易处理或投票。

### 区块链技术上如何工作？

阅读文章或教程理解 Blockchains 并不容易。徘徊在许多在线指南中，我发现了 Daniel van Flymen 写的[这篇文章](https://hackernoon.com/learn-blockchains-by-building-one-117428612f46)它和好奇心促使我去了解区块链如何真正起作用，并试图用 NodeJS 创建区块链。

#### 1.创建一个 logo

创建新项目的第一步是创建徽标。
它使一切变得真实。
因此我使用了 [Squarespace 徽标创建器](https://logo.squarespace.com)，结果如下：

![image](https://cdn-images-1.medium.com/max/1600/1*rwQfjbucSPr28duwb9x3Iw.png)

现在可以真正开始我们的项目了。

#### 从 Express 开始

为了方便创建 API 与区块链互动，我使用 Express 包直接启动了项目。使用 npm [express-generator](https://www.npmjs.com/package/express-generator)

```
npm install -g express-generator
 express ./chiccocoin
 cd chiccocoin
 yarn
```
可以在下面这个仓库中找到所有代码：

<a href="https://github.com/thecreazy/chiccocoin">
    <div style="float:left">
        <p>thecreazy/chiccocoin</p>
        <p>chiccocoin-Chicco Coin:Learn what is a blockchain by createing one github.com</p>
    </div>
    <img style="float:right" src="https://cdn-images-1.medium.com/fit/c/320/320/0*yyLqgsTZwkKm7fLT." />
</a>

#### 3.创建区块链

现在要创建我们的区块链。
重读区块链的定义，可以总结出绝对必要的功能是：

- newBlock: 创建一个新块
- newTransaction: 创建交易并将其排入队列以在下一次创建块时保存
- hash: 加密一个块
- lastBlock: 返回链上的最后一个块

所以我们可以用这个结构创建我们的 blockchain.js 文件：
<center>启动区块链的 JavaScript 类</center>

我们类的构造函数将创建两个重要的变量，chain 和 current_transaction。**Chain** 将按顺序包含我们所有的区块。**Current_transactions** 将包含下一次添加到块中的所有交易。

#### 3.1创建一个块

但是什么是块？块的表示是一个 JavaScript 对象，其中将包含：

- Index(索引)
- Timestamp(时间戳)
- List of transactions(交易列表)
- Proof(证明)
- Hash of the previous Block(前一个块的哈希)

<center>表示块</center>

这样区块链的想法就变得更清晰了。是一组连续的块（**index**）链接到前一个块，并使用加密（**previous_hash**）保护。块内的 previous_hash 是整个区块链的关键。它使我们能保证整个链条的安全和稳定。如果攻击者可以修改一个链的块，那么该块之后的所有哈希将是错误的。很明显它可以试着重新计算整个链条的哈希值，所以更多的区块更多的区块链比较安全。而且每个节点（本例中为 NodeJS 的实例）都有一个（相同的）区块链副本，这使得黑客几乎不可能同时修改所有副本。

所以在我们的例子中创建一个新块非常简单。
只需要在链中加一个新对象。函数会收到证明(一会我们讨论下它是什么)和前一个块的哈希，返回一个块。还会把所有未保存的交易添加到块中，并清理 **current_transactions** 变量。

##### 3.1.1 哈希函数

Chiccocoin 里面用到的哈希函数是一个简单的 SHA256，你可以使用你喜欢的哈希函数。重要的是给前一个块做哈希操作，所以我们会给序列化对象做哈希操作。

#### 3.2 创建一个交易

添加新交易的函数非常简单。给数组 current_transaction 加一项。交易是由发件人，收件人和金额组成的对象。它将是块内存储交易的挖掘函数。对于实用程序，我们将确保该函数返回将要保存的块的索引。

<center>区块链文件</center>

#### 4. 作用证明

通常，作用证明是一个函数或一个协议去阻止[拒绝服务](https://en.wikipedia.org/wiki/Denial_of_service)的攻击，但区块链用它来确定新块是如何创建的或挖掘自己。POW 的目标是发现解决问题的数字。这个数字必须是**难发现但易校验**的，像素数的计算，越难找到就越难，但要了解它是否一直是很平庸的。

为了挖掘 Chiccocoin，我们决定创建一个 c4ff3。POW 会是这样：

> 用 P 给前一个块做哈希操作，会得到一个以 c4ff3 开头的哈希。

例子：


```
c4ff3e9373e...5e3600155e860
```

这个函数有两个是必要的：

- **validProof:** 提供前一个 POW 和 P 来检查解决方案是否正确。
- **proofOfWork:** 一直循环知道找到解决方案

<center>作用证明函数</center>

#### 5. 服务 API

用 expressjs 启动区块链项目会非常简单。

创建三个 API：

- `/transactions/new` 在块中创建一个新交易
- `mine` 告诉服务器挖掘一个新的块
- `chain` 返回完整的区块链

我在`/middleware/chiccocoin.js`中创建了一个 Chiccocoin 支持类，它包含 API 所有必要的中间键并实例化一个新的区块链。

#### 5.1 链端点

这个端点非常简单。
只返回存储在区块链内的 ***chain*** 数组。

#### 5.2 交易端点

交易端点会检查传递的数据，调用区块链的 **newTransaction** 函数。未来，我们可以使用这个中间键检查实际发件人和收件人是否正确和/或交易是否可以完成。我们将使用 **newTransaction** 返回的信息通知用户哪个块的交易将被保存。

#### 5.3 挖掘端点

挖掘端点是 chiccocoin 实现的时候，必须做四件事：

1. 计算作用证明
2. 添加一个交易，可以获得一个硬币
3. 添加一个正在等待的交易
4. 通过将新块添加到链中来伪造新块

<center>chiccocoin.js 文件</center>

可以通过 curl 或 Postman 测试我们的 API。我添加了 ***postman_collection.json*** 来简化使用 postman。

### 结论

区块链背后没有比较新新或复杂的东西。Banally 开发者每天都会使用区块链而不会意识到它（git）。这是非常旧的观念，然而新观点正在改变对经济的思考，创造泡沫(在我看来)和安全经济的新可能。有能力控制交易发生的时间和方式是悄然发生但具有巨大潜力的事情。

我们创建的是一个简单的区块链，它只与一个节点一起工作，并且只有一个节点可以创建它（有基于此的加密货币）。当然，多节点的管理和围绕这个话题的所有问题都是同样有趣的。也许我们会就此做第二篇文章，在仓库中加 star 来获取更多更新吧。

<a href="https://github.com/thecreazy/chiccocoin">
    <div style="float:left">
        <p>thecreazy/chiccocoin</p>
        <p>chiccocoin-Chicco Coin:Learn what is a blockchain by createing one github.com</p>
    </div>
    <img style="float:right" src="https://cdn-images-1.medium.com/fit/c/320/320/0*yyLqgsTZwkKm7fLT." />
</a>

我希望这篇文章能促使你看看什么是比特币交易，并让你对区块链真正做的事感兴趣。










