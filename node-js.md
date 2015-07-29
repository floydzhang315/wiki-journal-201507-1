# 为什么是 Node.js ? 什么时候使用 Node.js ?  -- 王韬懿

![0](images/why-nodejs.png)

文章翻译：[王韬懿](https://github.com/noprom)

发表时间：2015 年 7 月 03 日

原文作者：Sachin FromDev

文章分类：后端开发

## 关于本文

对于服务器端开发语言来说，Java 和 PHP 可能是现在的企业用的最多的了。但是，Node.js 作为一门 Javascript 里面的服务器端语言在高并发方面的表现却不亚于用 Java 开发的企业级应用。国内国外的一些大型公司都逐渐的在将自己的应用使用 Node 构建，淘宝里面的一些模块就是很好的例子。本文将要介绍 Node.js 的一些基础知识和应用场景。


## 文章内容

### 学习什么是 Node.js ? 为什么它会这么受欢迎 ？你可以在哪些地方用到它？

想一下为什么 Node.js 越来越受欢迎了？什么时候使用 Node.js ？这篇文章有你想要的所有关于 Node.js 框架和最好的在社区实践的一切细节。

Node.js 通过用单线程取代多线程，事件驱动 I/O 来解决了高并发，取代了 Java 平台的标准。在这篇文章中，我们将讨论 Node.js 并且会阐明为什么事件驱动并发会如此的受欢迎，甚至包括一些核心的 Java 开发者。

我们等会儿会向你介绍一些 Node.js 最好的实践来让你使用 Node’s Express 框架，Mongolian DeadBeef 和 MongoDB 来做一个即时的，可扩展的，并且持久化的 web 应用。

### 一些历史背景

在过去的一些年里面， JavaScript 已经成为了一个“黑暗骑士”，有点像是 web 开发的传奇。但是事实却让很多过去说 JavaScript 是一门 “玩具语言” 的 web 应用开发者感到惊讶。

虽然有很多知名的语言，但是 JavaScript 作为标准的，浏览器中的脚本语言却让它保持主流地位。对于网站客户端的开发，它可是全球用得最广泛的语言。

JavaScript 额外的在服务端有一席之地，并且这方面正在发展。虽然之前 JavaScript 在服务端有一些企图，但是却没有一个像 Node.js 或者 Node 这样吸人眼球。

### 什么是 Node.js

Node.js 是一门开源的，跨平台开发的服务器端和网络应用。 Node.js 应用使用 JavaScript 开发的，并且这些应用可以在  OS X, Microsoft Windows, Linux, FreeBSD, NonStop 和 IBM 的 Node.js 运行环境里面。

### 目的

为了帮助开发者构建适应性强的系统， Node 是一个为开发者打开了一个全兴领域的用 JavaScript 写的服务器端开发环境。对于一些 Java 开发者来说，Node 最大的好处就是它的简单的可以解决程序高并发问题的方法。

虽然 Java 平台持续的开发它的解决高并发的方式（极大的在 Java 7 和 Java 8里面得到了提高），但是重要的事实就是它满足了一个特殊的需求，并且很多 JavaScript 开发者都在投入它的怀抱。

就像客户端的 JavaScript 脚本，在 Node 环境下面的服务器端脚本在工作上是如此的棒，并且它工作在很多 Java 开发者工作的领域。

在这篇文章中，我们首先会以一个是什么让 Node 如此杰出的结构化的大纲开始，在那之后，我们会展示可以迅速开发一个可扩展的 web 应用的行业标准，这个行业标准影响了 MongoDB 的信息产业。作为读者，你可以发现 Node 是如此有趣，并且开发一个可工作的 web 应用需要多少时间。

### 为什么 Node.js 是如此流行？

所有的科技巨头热爱 Node.js 有很多原因。越来越多的应用正在用 Node.js 开发，并且正在被行业所采纳。这里有一些明显的 Node.js 流行的原因。

- 易于学习

    Node.js 是一个非常容易学习的环境。它是基于 JavaScript 构建的，因此开发者主要集中在学习 JavaScript 编程上面。
    
    很多 JavaScript 开发者可以简单的进行自学。与此同时，这里有很多为初学者准备的[好的 Node.js 图书。](http://www.fromdev.com/2014/03/node-js-books-top.html)
    
    对于那些不懂 JavaScript 的可以在 [JavaScript 教程](http://www.fromdev.com/2013/08/Javascript-Tutorials.html) 免费学到。初学者可以参考 [好的 JavaScript 图书](http://www.fromdev.com/2012/10/JavaScript-Books-Developer-Must-Read.html)。

- Node.js 支持跨平台开发

    Node.js 可以在主流操作系统上运行，包括 Linux, Windows 和 Mac OSX。一个简单的安装就可以适应所有环境。
    
- Node.js 的可测量性和高性能

    一个快速的采纳 Node.js 的主要的原因就是因为它在生产系统上面的性能。很多高流量的系统就是用 Node.js 写的。像 Paypal 和 Walmart 这样的公司已经宣称他们的 Node.js 应用在高流量的时候的表现非常好。
    
- Node 的事件驱动并发

    Node 是一个基于谷歌的 V8 JavaScript 引擎的事件驱动I/O环境。 在运行之前 Google V8 真的与 JavaScript 放进本地机器码，带来了惊人的快速运行时执行 － 一些不适通常与 JavaScript 相关联的。因此，Node 可以让你迅速的构建快速并且支持高并发的网络应用。
    
    事件驱动 I/O 对于 Java 开发者来说可能有一些遥远，但是这不是最新的。不是我们在 Java 里面使用的多线程编程模型，Node 解决高并发的方式是单线程，只需要额外的事件循环。这让 Node 可以无阻塞或者异步 I/O。在 Node 里面，经常阻塞的调用，比如说，等待数据库查询结果，不会发生。
   
   Node 需要一个回调，而不是为了完成一个高耗费的任务。当一个数据被返回的时候，联合的回调会被非高并发的调用。
   
   高并发会在 Node 程序里面起作用。如果我需要在 Java 平台上面运行过去的数据，我会考虑我的决定的复杂度和延长的方法 -- 从通常的线程到 Java NIO 里面的刷新的类库，甚至提高并且重新设计的 java.util.concurrent 包。
   
   虽然 Java 可以实现高并发，但是它在代码上面非常难理解。Node 的回调系统，是与语言结合在一起的，你并不需要像 synchronized 这样特殊的构造器来让它工作。Node 的并发模型是惊人的简单，这让更大范围的开发者可以接触到它。
    
- 更好的能力和保持力

    虽然你可以找到成千上万的 Java 开发者，但是 Node.js 的开发者并不匮乏。Node.js 在[最好的自由职业者网站](http://www.fromdev.com/2015/01/freelance-jobs-websites.html)上面有很多。
    
- 更强大的生产力

    因为 Node.js 是既用于服务器端和客户端的开发，它的开发也更加快速和稳定。一个 JavaScript 开发者可以理解并且从头到尾的开发一个应用，而不需要担心服务端与客户端之间的复杂的一层关系。
    一个基于 Java 的应用可能仍然需要一个 JavaScript 开发者去丰富用户界面，并且从更深层次上说可能需要 JavaScript 开发者和 Java 开发者之间的沟通交流。
    
- 活跃的社区

    Node.js 社区非常活跃，并且快速帮助在流行的论坛和讨论组里面总是可以获到。[Stackoverflow](http://stackoverflow.com/questions/tagged/node.js?sort=frequent) 有超过 80K 的问题，并且很少有问题是没有被回答的。
    
- 很多有用的开发者工具

    因为 Node.js 主要是 JavaScript ，因此为它找开发工具非常简单。有很多支持 JavaScript 的 IDE 和开发者工具可以用来写 Node.js。
    JavaScript 可以被浏览器解析，因此很多[基于云服务的 JavaScript IDE](http://www.fromdev.com/2015/01/websites-to-test-javascript-code.html) 可以用来在线开发和调试 Node.js。

- 简单的 Node.js 主机 

    虽然你可以选择一个[专注的主机服务商](http://www.fromdev.com/2015/05/dedicated-server-benefits.html)并且可以简单的搭建你自己的 Node.js 环境，但是有很多你可以简单的安装 Node.js 环境的[主机提供商](http://www.fromdev.com/2014/06/best-web-hosting-companies.html)。这些提供商已经有一个基本的为 Node.js 应用的安装步骤，这可以让你快速开始你的业务。
    这里有一个[支持 Node.js 的主机提供商](https://github.com/joyent/node/wiki/Node-Hosting)列表。

### 为什么选择 Node.js?

Java 平台在大企业开发里面的高并发服务占据主导地位，这一点有可能不会改变。像 Netty (和 Gretty)的框架，像 NIO 和 java.util.concurrent 这样的类库，已经在 JVM 的高并发部分占据地位。

关于 Node 很奇特的一点就是它是一个专门为解决同步编程困难的流开发环境。Node 的事件驱动编程模型告诉你你需要为了同步工作而与其他额外的类库打交道，这对于开发者来说是一件振奋人心的消息。

### 我们在哪里可以用到 Node.js?

在很多应用场景 Node.js 可以工作得比传统的服务器端编程语言要好，比如 Java 或者 PHP。Node.js 主要的好处就是客户端与服务器端之间两端的连接。这让与服务器端有很少交互的丰富的客户端应用使用 Node.js 成为一个流行的选择。

如果你想创建一个在页面上面有很多种类型的数据格式和组件的面板应用，Node.js 可能是一个很好的选择。一些 Node.js 运行的很好的实用的应用如下。

- 展示一个股票市场的面板
- 监控开发管理的面板
- 一个在网页上的直播网络目标系统
- 生产系统监控面板
- 聊天室应用
- 基于 JSON 和 NOSQL 数据库的应用
- 需要和移动设备和原生应用交互的应用
- 需要和互联网进行交互的应用

### 我们在哪些地方不需要使用 Node.js?

就像其他事物一样，Node.js 并不是适合解决你的所有的问题。下面的一些系统使用 Node.js 可能并不是一个好的选择。

- 需要大量计算和大量消耗 CPU 的应用
- 与关系型数据库（如 Oracle, MySQL 或者 Postgress）交互的应用

### 总结

Node 的 JavaScript 的语言结构当然会让你远离键盘。只需要一点代码，你就可以制造一个快速的，通用的 web 应用。

你可以在 Java 平台上面做到那一点，但是你需要更多行的代码和额外的类库和开发。并且，以防止你在研究其他编程环境时被惹怒，请不要这样以为：你知道一些 JavaScript 的话，Node 就非常简单。我敢打赌你会这样做。

 > 更多IT技术干货: [wiki.jikexueyuan.com](wiki.jikexueyuan.com)   
> 加入极客星球翻译团队: [http://wiki.jikexueyuan.com/project/wiki-editors-guidelines/translators.html](http://wiki.jikexueyuan.com/project/wiki-editors-guidelines/translators.html)   

> 版权声明：   
> 本译文仅用于学习和交流目的。非商业转载请注明译者、出处，并保留文章在极客学院的完整链接   
> 商业合作请联系 wiki@jikexueyuan.com   
> 原文地址：[http://www.fromdev.com/2015/07/why-node-js-popular.html](http://www.fromdev.com/2015/07/why-node-js-popular.html)   
