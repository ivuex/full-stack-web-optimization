# commonOptimizationIndicator

## 1. 常用的性能优化指标
>   对于网站性能指标最直接的印象可能是网站的响应速度，
>   因为这是访问者最直观真的体验。
>   网站访问的过程从用户输入网站域名开始，
>   通过DNS解析找到目标服务器，
>   目标服务器收到请求后执行服务器及数据库等一系列操作，
>   并将响应数据经过互联网发送到用户浏览器中,
>   最终由浏览器处理响应数据并完成网页的渲染呈现。

### 1.1 网页的资源请求与加载阶段

让我们来看一看用户访问网站发送资源请求的全过程,如图 1.1所示：

![图1.1.1 网站资源访问过程](http://optimization.ivuex.tech/img/chromeDevToolsTiming.png)

*图1.1.1 网站资源访问过程*

　　从图中可以看到，网页的访问请求过程，包含了建立连接和请求响应两个阶段。由于HTTP协议是建立在TCP协议上的应用层协议，上述浏览器中建立链接阶段，及客户端到服务器之间建立TCP连接。

　　在建立连接的阶段，Queueing和Stalled表示请求队列以请求等待的时间。由于在HTTP协议上，Chrome浏览器只允许每个源拥有6个TCP连接，因此如果请求处于正在排队或者等待状态，说明请求需要等待以释放不可用的TCP连接，因此如果请求处于正在排队或者等待状态，说明请求需要等待一时方便库尔用的TCP套接字，或者由于该请求的优先级低于关键资源而被渲染引擎推迟加载。DNS Lookup则是执行DNS所用的时间。页面上的每一个新域都需要完整的往返才能执行DNS查询。 Initial Connection和SSL则包括TCP握手重试和协商SSL以及SSL握手的时间。
　　在请求响应阶段，Request sent是发出网络请求所用的时间，通常不超过1ms。Waiting(TTFB)等待初始响应所用的时间，也称为等待返回首个字节的时间，该时间将捕捉到服务器往返的延迟时间o

　　由此可见，为了实现网站的快速响应，首先需要考虑的就是减少资源访问及加载阶段所消耗的时间。前面已经说过，在使用HTTP 1.0/1.1 协议时， Chrome 会将每个主机强制设置为最多6个TCP连接，因此可以通过划分子域的方式，将多个资源分布在不同子域上用来减少请求队列的等待时间。然而，划分子域并不是一劳永逸的方式，多个子域意味着更多的DNS查询时间。通常把域名拆分为3到5个比较合适。

　　另一方面，HTTP是一个无状态的面向连接的协议，即每个HTTP请求都是独立的。然而无状态并不代表HTTP不能操持TCP连接，Keep-Alive正是HTTP协议中保持TCP连接非常重要的一个属性。在HTTP 1.1 协议中，Keep-Alive默认打开，使得通信双方在完成一次通信后仍然保持一定时长的连接，因此浏览器可以在一个单独的连接上进行多个请求, 有效地降低建立的TCP请求所消耗的时间。但需要说明的是，就算是在HTTP1.1版本中,Keep-Alive也不能保证浏览器和服务器之间的连一定是活跃的，所以不应该然程序依赖于Keep-Alive保持连接的特性, 否则会有意想不到的后果。

> 注意： 由于浏览器自身的缓存DNS，且在缓存中查询可以减少DNS的查询时间,所以实际应用中划分子域对性能的影响很难进行量化，这也是提倡在前段领域使用公共CDN服务器托管类的原因之一。另外，HTTP 2.0 协议提供了单个TCP连接多路复用的能力，极大地提高了Web应用的性能，而本节所讲述的任然是基于传统的HTTP 1.0/1.1 协议下的性能优化。

### 1.2　网页渲染阶段
　　实际上，在 1.1节的请求响应过程中,仅仅完成了网页的资源请求与加载。站在前段开发性能优化的角度，网站响应速度的快慢，不仅限于万占资源加载速度，网页的渲染速度和前段JavaScript的运行速度同样直接影响着访问者的用户体验以及网站的整体性能。
　　重度交互的网站应用中，体现的愈发充分。我们可以到Chrome浏览器DevTools中看一下页面渲染的性能优化, 如图13.2所示。

![图 1.2.1　网页访问个阶段时间消耗](http://optimization.ivuex.tech/img/timeOfEachAssetsAccessStages.png)

*图 1.2.1　网页访问个阶段时间消耗*

　　图中的Rendering以及Painting时间，就是浏览器在页面渲染过程中的时间消耗。在对浏览器渲染过程就行优化之前，再次了解一下浏览器是如何渲染网页的。

　　首先，浏览器将从服务器获取的HTML文档构建成文档对象模型DOM (Document Object Model), 与此同时浏览器会下载文档中引用的CSS与JavaScript文件。CSS经过解析构成层叠样式表模型CSSDOM(CSS Object Model), 而Javascript则会交给Javascript引擎执行，紧接着，文档对象模型DOM与层叠样式模型CSSDOM将构建渲染树(Render Tree), Render Tree 上的节点称之为RenderObject, 因此Render Tree上的每个节点对象递归检查是否需要创建RenderObject,　并根据DOM节点类型创建RenderObject节点,　动态加入DOM元素。由于页面的非可元素并不会形成 RenderObject, 因此　Render Tree上的节点并不与DOM　Tree一一对应, 为了方便处理定位、Z轴排序、页内滚动等问题,　浏览器并不以Render Tree为基础直接进渲染，而是依据RenderObject生成新的Render Layout Tree, 浏览器渲染引擎将会遍Render Layer Tree, 访问每一个RenderLayer, 再从遍历从属于这个RenderLayer的RenderObject，将每一个RenderObject绘制出来, 执行Composite合并RenderLayer并最终呈现给用户。

### 那么如何才能提高浏览器渲染的性能？

　　首先需要注意的是，浏览器在加载JavasCript节点后，会交由JavaScript引擎执行,如果JavaScript对DOM树进行读写操作,　则会影响DOM树的构建,　从而阻塞浏览器的渲染过程, 解决此问题通常的做法是将 JavaScript 放在页面底部, 或者通过异步的方式加载JavaScript。另外，过与复杂的CSS嵌套规则，会影响CSSDOM的生成, 导致渲染的时间被延长。在浏览器首次渲染以后，页面上的元素还可能不断地被重新布局和绘制。对DOM节点进行添加和删除操作、通过JavaScript修改一些CSS属性、修改元素的display样式属性、移动或对节点添加动画、滚动或调整浏览器窗口大小等, 这些操作都会导致浏览器的重新布局以及重新绘制,而修改元素的visibility、背景色和前景色等也会导致重新绘制。如果处理不当，这些动作可能会产生性能问，产生不好的用户体验。如图1.3所示，诠释了浏览器重新布局以及重新绘制的过程。

![图 1.3.1  浏览器在重新布局以及重新绘制过程中的性能消耗](http://optimization.ivuex.tech/img/eventLogPanelInChromDevTools.png)

　　在上述过程中, 最需要避免的是过多的重新布局, 重新布局必然会导致后续的重新绘制以及合并。然而有一些特别的熟悉你如opacity或者transform可以在不同的层中单独绘制。对这种属性的访问, 并不会导致重新绘制，而直接执行了合并图层Composite Layouts。由于图层合并Composite Layouts这个过程一般发生在GPU的硬件渲染中，通常会非常高效，并可以减少渲染过程中的性能消耗。

---
tobe enhancing

1.3 JavaScript脚本的执行速度
### JavaScript 脚本的执行速度
　　JavaScript 脚本的执行速度是一个涵盖知识点非常多的话题。前面的章节已经介绍了JavaScript的性能测试工具Benchmark。读者可以通过jsPerf提供的基于Benchmark运行的共享测试用例,来比较不同JavaScript代码段的性能和执行速度，本节就不做过多赘述了。

tobe enhancing ...
---