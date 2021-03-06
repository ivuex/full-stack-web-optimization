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


## 2. 依有效的Yahoo性能优化法则

　　针对如何提高Web性能，yahoo提出了许多优化建议，这些优化建议悲观大开发者所爱那，并称为"雅虎法则"或"雅虎军规"。时至今日，雅虎法则已经从最初的14个性能优化点发展未包含有内容优, 服务器、Cookie、CSS、JavaScript、图片和移动应用7大分类共计35条优化建议的一整套提高Web性能的最佳实践。

　　接下来花点时间了解一下雅虎法则中关于前端性能优化的的几个要点。

+ **减少HTTP请求**
　　这是最为重要的一条优化法则。在上一节中，我们已经详细探讨了浏览器资源请求的过程。由于在HTTP请求也可以降低服务器负载，减少病发病提升服务器的处理能力。减少HTTP请求的手段有很多,比如合并CSS与JavaScript文件、使用CSS Sprites、内敛图像"data:URL scheme"等。

+ **压缩CSS和JavaScript代码**
　　压缩CSS与JavaScript代码的体积,精简不必要的空格、换行、缩进等。代码的字节数减少，代码对应的下载时间也会随之减少。

+ **去除重复引用的脚本**
　　重复脚本在Internet Explorer浏览器中会导致多余的HTTP请求,也会带来不必要的运算，降低网站的性能。

+ **可缓存的AJAX**
　　AJAX 经常被提及的一个好处就是异步性。该技术异步地从服务器传输信息，为用户带来即时反馈。但使用AJAX并不保证用户不会等待异步请求，但使用AJAX并不保证用户不会等待异步请求,通过Expire或者Cache-Control头来实现缓存，提升网站性能。
+ **延迟加载非必要脚本**
　　页面上有些内容并不需要立刻加载。非首屏的图片资源，需要经过用户操作才能呈现的非可见元素等都可以惊醒推迟加载。

+ **预加载**
　　预加载看起来和延迟加载恰恰相反，实际上预加载是指在浏览器空闲阶段遇限价在将来用户可能访问到的内容,从而提高页面的即时响应能力,优化用户体验。

+ **减少DOM元素数量**
　　DOM元素过多意味着需要加载更多的数据,在使用JavaScript遍历DOM时更加低效，也意味着执行布局和绘制时产生更多的性能损耗。

+ **减少DOM元素次数**
　　DOM被认为天生就慢，通常需要将经常访问的DOM对象缓存，避免过多的使用JavaScript修改页面布局。

+ **避免使用iframe**
　　即使引入页面内容为空，iframe也需要消耗时间下载, 会阻止页面加载，并且缺少语义。

+ **优化图像**
　　设计师提供的图片大多数都存在压缩空间，或者可以GIF格式转换为PNG格式。

+ **优化 CSS Sprites**
　　CSS Sprites中，图片水平排列较之垂直排列体积更小，可以将颜色接近的组合在一起。不要在Sprites中留有较大的空隙。空隙虽然不会增加文件大小, 但是对于用户需要跟多的内存来把图片解压为像素地图。

+ **不要在HTML中缩放图片**
　　在HTML中设置的图片长宽小于实际下载图片长宽,很显然实际加载的图片比需要的图片体积更大。

+ 减少Cookie体积

  - Cookie信息通过HTTP文件头在服务器和浏览器之间进行传递,因此保持Cookie尽可能小,可以减少用户的数据传输时间。

　- 以上列了雅虎法则中前端优化最常用的几条,更多的内容可以查阅雅虎开发者官方网站。通常一个网页从发起服务器请求到实现对最终用户的展现，在客户端所花费的时间大约会占到总时耗的80%，因此客户端的性能优化显得尤为重要。这也是成为优秀前端工程师必须要掌握的基础知识。

## 3 性能优化工具使用实战

　　在网站进行优化之前，首先需要进行性能分析。目前主流的性能分析工具, 大多会提供浏览插件以及在线分析等几种方式。其中比较有代表性的性能分析工具有: YSlow(By Yahoo), PageSeed(By PageSpeed), WebPageTest(By Google)。
-- reduced --

## 4. HTTP 协议头缓存实战

　　提供更好的用户体验一直是Web开发者最求的目标。一个引用大量静态资源的Web页面, 资源的加载速度是影响页面加载耗时的最大因素。很多固定静态资源，如图片、脚本、样式文件等若都可以通过缓存保存在客户端, 不必每次都从服务器请求, 将节省大量请求时间。那么，客户端和服务端通过怎样的方式，才能确定客户端知道是否需要使用缓存文件。本节将会介HTTP缓存的具体细节。

### 4.1 客户端缓存流程

　　现代浏览器会根据HTTP协议中的缓存, 实现本地缓存功能。在请求一新的文件时，浏览器发送HTTP请求到服务端。接到服务端的响应后, 浏览器会将请求的资源储存在本地，留作以后使用。

　　服务端响应中，会带有文件相关的缓存策略，高速浏览器文件是否需要缓存以及缓存何时过期等信息。当浏览器再次请求文件时,会先判断缓存中是否有相应的文件以及是否过期，未过期则直接从缓存中读取文件, 不会在向服务器发送请求。

### 4.2 缓存协议内容
　　HTTP头中关于缓存相关的属性, 主要由以下几种。

1. Expires:　指定缓存过期时间，是一个绝对时间，但受客户端和服务端时钟和时区差异的影响。

2. Cache-Control:　比Expire策略更详细,优先级比Expires高，其值可以是以下五种情况。

   -   no-cache: 告诉客户端不使用缓存。
   -   no-store: 告诉客户端不要缓存响应。
   -   public: 缓存响应, 并可以在多用户间共享。
   -   private: 缓存响应, 但不能在用户间共享。
   -   max-age: 缓存在指定时间(单位为秒)后过期。

3. Last-Modified-Since:　指定响应资源的最后修改时间。如果响应头中包含Last-Modified,　再次请求时通过If-Modified-Since将最后修改时间告诉服务端，服务端判断文件是否有过修改，再决定返回新内容还是通过HTTP状态吗304告诉客户端使用缓存。

4. ETag/If-None-Match:　区别内容的唯一标识，需要配合Cache-Control使用。当文件最后修改时间发生变化，但文件内容并无改变时，也应该使用缓存，如果响应头中包含ETag，再次请求是通过If-None-Match将内容标识高速服务端，服务端比较内是否有改变后，再决定返回新内容还是通过HTTP状态码304告诉客户端使用缓存。

　　使用浏览器的调试工具或者通过代理工具Fiddler等，可以查看请求和相应的头部内容。在Chrome浏览器中，打开调试窗口切换至Netword选项，可以看到客户端发送的请求列表。初次请求时，，服务端返回状态码200；服务器判断文件未修改返回状态码304；max-age或Expires未过期时，客户端直接读取本地缓存，如图1.4所示。

![图 4.2.1　网络请求中的缓存状态](http://optimization.ivuex.tech/img/cookieStatusInNetworkRequest.png)

HTTP头中的缓存状态设置，如图 1.5所示。

![图　4.2.2　HTTP头部中的缓存设置](http://optimization.ivuex.tech/img/cookieSettingsInHTTPHeader.png)

### 4.3 缓存过程实例分析

　　如果通过Node.js在本地启动一个服务器测试HTTP缓存功能。比如以"Cache-Control:max-age"为例，为了方便测试缓存过期后的效果，将max-age的值设置为 5s,　服务端代码如下：

```
response.setHeader('cache-Control', 'max-age: 5');
```

　　在Chrome中访问本地(比如3005端口)实例uri：　http://localhost:3005/cachedemo，接着 5s 内在控制台通过 Fetch Api 再次请求，5s 后再次发送请求。
```
　　fetch('http://localhost:3005/cachedemo');
```
在Network下观察三次网络请求，如图 1.6 所示：
![图 4.3.1　测试 max-age ](http://optimization.ivuex.tech/img/maxAgeTest.png)

从图中可以看到：

- 第一次请求时没有缓存，因此直接从服务器访问资源；
- 第二次请求时，浏览器本地已经存在缓存并未过期，因此直接从本地磁盘读取了缓存内容；
- 第三次 5 秒后的再次请求，因为本地缓存已过，浏览器重新从服务器请求资源。

## 5. 资源按需加
### 5.1 基于RequireJS的按需加载
　　RequireJS是一种AMD(Asynchronous Module Definition 即异步模块定义) 的实现，采用异步加载模块，因此模块的加载不会影响后续代码的运行，官网地址为 http://requirejs.org/ 。AMD通过require函数加载模块，接受两个参数，实例代码如下:

```
　　require([module], callback);
```

- module:　所有要加载的模块，数组类型。
- callback:　模块加载并运行完后执行的回调函数

　　在实际开发中，开发者需要在页面上引入 RequireJS 的加载器代码和一个主 JavaScript 文件用来加载依赖模块，接受两个参数，实例代码如下：

```
requirejs.config({
    baseUrl: 'js/modules', //配置
    paths: {
        "module-a": "a.js",
        "module-b": "b.js",
        "module-c": "c.js"
    }
})
```

　　使用require函数引入三依赖模块，代码如下：

```
// 可以通过灰调函数的参数调用木块的能力
require(['module-a', 'module-b', 'module-c'], function(a, b, c) {
    // 具体业务代码
});
```

　　在每个模块的JavaScript文件中，开发者首先需要定义模块，AMD规范提供了一个 define 函数，如果一个模块不依赖其它模块，可以直接进行定义，代码如下：
```
define(function () {
    var add = function (x, y) {
        return x + y;
    }
    return {
        add: add,
    }
});
```

　　如果依赖其他模块则可以把以来模块数组作为第一个参数传入，代码如下：

```
defind(['module-c'], function(1) {});
```

　　RequireJS 可以帮助开发者异步加载 JavaScript 代码，解决了模块之间的依赖关系，提升应用的整体质量和性能。

### 5.2 基于 Webpack 的按需加载
　　CommonJS 规范虽然本身采用同步加载模块，但也提出了 Modules/Async/A 规范，定义了一套 require.ensure 用于处理异步加载。require.ensure 会将模块下载下来，Webpack 作为一个模块加载器同时也是一个打包工具，所以开发这不需要特意的去定义模块，Webpack 会使用"Code Splitting"技术实现分批打包和按需加载。假设依旧存在 a、b、c 三个 JavaScript 文件，在 app.js 文件中使用以下代码：
```
require('./module/a');
require.ensure(['./module/b'], function(require){
    require('./module/c');
});
```
　　在最后的打包工具中，会有一个 bundle.js 和 0.bundle.js。bundle.js 包含 Webpack 信息、模块 a 和 app.js 的内容。0.bundle.js 包含 Webpack 信息、模块 a 和 app.js 的内容。0.bundle.js 包含 模块 b 和模块 c 的内容。最后开发者只需要在页面中引入 bundle.js 文件，则 0.bundle.js 文件会被动态引入。
　　虽然 b.js 和 c.js 被打包在了一个文件，但只有 c.js 的内容被执行，如果需要运行 b.js 仍需要手动输入 "require('./module/b')", 这种模式虽然需要手动运行模块，但是可以控制以来模块的执行顺序，而在RequireJS 中，以来模块的加载和运行时不固定的。

> *提示：* Modules/Async/A 规范，参考地址为 [http://wiki.commonjs.org/wiki/Modules/Async/A ](http://wiki.commonjs.org/wiki/Modules/Async/A)

### 5.3 图片懒加载
　　前端页面加载中另一个对用户体验很大的影响因素是图片，图片往往是页面上体积最大的资源。当页面中粗在太多图片时，页面加载时间较长。懒加载的原理是通过监听页面滚动事件，代码如下：




```
<img class="lazyload" data-src="图片的真实路径">
```

　　这里的 "data-src" 属性储存的就是真实路径，在页面渲染时，因为浏览器不会处理该自定义属性，所以不会自动加载图片。接着，监听页面的滚动事件，代码如下:

```
var imgList = Array.prototype.slice.call(document.querySelectorAll('img')); // 获取所有图片
function loadImage () {
    for (var i = 0; i < imgList.length; i++) {
        var el = imgList[i];
        if(isShow(el)) {
            el.src = el.getAttribute('data-src');
            imgList.splice(i, 1);
        }
    }
}
function isShow(el) {
    // 获取图片在屏幕中图片顶部与屏幕顶部的距离、图片底部与屏幕顶部的距离、
    // 图片左边框与屏幕左侧的距离、图片右边框与屏幕左侧的距离、及图片的宽高
    var rect = el.getBoundingClientRect();
    // 如果 rect.right 或　rect.bottom 小于 0, 意味着图片整体为进入屏幕
    // 如果在大于 0 的情况下， react.top　和　rect.left 比较　innerWidth 和　innerHeight
    var isAppear = rect.right > 0 && rect.left > (-1) * window.innerWidth &&
                rect.bottom > 0 && rect.top > (-1) * window.innerHeight;
    return isAppear;
}
```

　　这里提供了一个最简单的图片懒加载方案。真实的业务尝尽中还需要考虑用户下拉速度、液面高度的固定性、iScroll 等第三方插件库的使用情况。笔者同时也推荐开发者使用一些开源的懒加库，以方便业务开发。
> *提示:* iScroll 是　Matteo Spinelli 开发的旨在解决移动端浏览器的区域滚动问题，使用原生 JavaScript 编写，不依赖于任何第三方框架，最新版本为 iScroll5。官网地址为 http://iscrolljs.com/　。

## 6 不同网络类型的优化实战

### 获取网络类型

　　web APIs　在 Navigator 接口中新增 connection 属性来获取网络状态。开发者可以通过 navigator.connection.type 获取网络类型，包括 unkown、Eherne、WIFI、2G、3G、none。理想情况下可以利用这个信息对资源进行优化，但这个 API 属于一个实验性质的属性，[支持的浏览器的情况](https://developer.mozilla.org/en-US/docs/Web/API/Network_Information_API) 如图 13.6 所示:

![移动端　Network Information API　浏览器支持情况](http://optimization.ivuex.tech/img/mobileBrowserCompatibilityOfNetwork_Information_API.png)

![PC端　Network Information API　浏览器支持情况](http://optimization.ivuex.tech/img/desktopBrowserCompatibilityOfNetwork_Information_API.png)

　　可以看出大部分浏览器都不能很好地支持该属性，所以通常需要开发者主动考虑用户的弱网情，在代码层面上进行优化，或者通过混合开发模式在APP中通过接口获取用户的网络类型。

###　6.2 弱网图片优
　　图片通常是一个页面上最小号网络资源的内容，如今很多手机已经配上了高分辨率的屏幕，因此在实际开发中，开发者会使用高分辨率的图片缩放以保证清晰度。但这样做的同时也增大了图片资源的体积，所以通常在 Web 开发环境下使用两倍分辨率即可，在 APP 中甚至可以根据网络环境使用根据网络环境使用更低分辨率的图片。
　　在前端开发中，开发者会使用雪碧图来减少对资源的请求数量。这种方式正确与否要视体情况而定。将高分辨率图片整合成一张雪碧图是不合理的，因为这样的图片会需要很长的加载时间，从而影响用户体验。正确的做法是仅将小图标整合到雪碧图，并控制每张雪碧图的体积，如果超过了上线，则整合第二张雪碧图。在雪碧图合成重要使用合理的排放策略，甚至手动合成，比便有一些图标在合成过程中产生大量留白，造成资源的浪费。

> *说明：*　雪碧图，即 CSS Sprite, 将页面中的图标或者背景图片合并到一张大图中，通过控制 CSS 的 background-position 属性确定图片呈现的位置。

　　图片格式也是优化的重点。如果不需要透明图层，比如背景图，则避免使用 PNG 格式。打尺寸图片需要选择JPG格式，同时加一定的压缩比例，这样能够把体积件收到原来 PNG 格式的几分之一。 开发者还可以在 APP中使用 WebP 格式的图片，兼顾实现效果和体积。
　　另外还可以使用字体图标。比如，需要产生一定动画效果的图标至少需要两张图来实现，然而使用字体图标则可以通过 CSS来装饰。同理，一些简单的 GIF 动画，如 "loading" 效果图也可以使用原生 CSS 实现，不但减少了网络请求数量，同时还提升了性能。

### 6.3　若罔缓存优化
　　页面端的的数据请求也是一个对弱网命该的优化点，尤其在单一应用中，可能会同时拉去多个接口的数据。在一些 2G 网络下，页面会长时间处于加载数据的状态。遇到这种情况，卡法这可以选择在内存中缓存请求数据。例如，入口页为 A 页面，依赖数据源 a, 则可以在代码中定义一套缓存，示例代码如下:

```
var store = {
    a: {
        url: '', // 请求的地
        type: 'GET', //　请求的类型
        param: '', //　请求的参数
        cache: {}, // 缓存请求的结果
    }
```

　　当首次访问 A 页面时，通过调用 store.a 获取对应 A 页面请求参数信息，在成功获取数据后进行数据缓存。当视图切换回到 A 页面时，先判断是否存在 store.a 对象的 cache 属，如果存在数据则读取并返回，不进行网络请求。此外，还可在应用中默认进行网络请求，设置请求的超时时间。在请求 complete 事件中，假设发生请求超时，则读取缓存数据渲染页面，部分实例代码如下:

```
complete: function(XMLHttpRequest, status) {
    if (status == 'timeout') {
        var data = store.a;
        // 下面处理业务代码
    }
}
```

　　这种混存策略需要注意的是，如果存在其他修改当前缓存的可能，则需要在发出请求成功后的回调函数中保证缓存的更新。
　　除了内存中的缓存策略以外，HTML 5 还提供了 localStorage 作为客户端缓存方案，[localStorage 在现代浏览器中基本都得到了支持](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)，如图 6.3.1 和 图 6.3.2 所示。

![图 6.3.1　移动端　Web　Storage 浏览器至此情况](http://optimization.ivuex.tech/img/mobileCompatibilityOfWeb_Storage_AP.png)

![图 6.3.1　桌面端　Web　Storage 浏览器至此情况](http://optimization.ivuex.tech/img/desktopCompatibilityOfWeb_Storage_AP.png)

> *注意:*　localStorage 缓存数据的储存类型为字符串格式，所以在存储对象字面量时需要先通过 JSON.stringify 方法处理转换，在之后的使用中调用 JSON.parse 重新转换为原对象数据结构。

## 7. 优化案例：　Nginx 配置 Combo 合并 HTTP 请求
　　Nginx 是一个开源高效的 HTTP 服务器，通过简单的配置能够提供丰富的功，最重要的是，他对服务器的配置要求极低。Nginx 选用了有事驱动的异步非阻塞模型，而不是传统的依赖多线程来响应，因此，在同等资源的情况下能够提供极大的并发请求。对于前端开发者来说，选用 Nginx 是一个主流并且高性价比的选择。
### 7.1 安装 Nginx 和文件合并模块
　　要玩转 Nginx，首先推荐使用 Linux 操作系统的服务器来搭建服务，一台普通的云服务器即可。文件合并模快选择开源插件 nginx-http-concat, 操作系统为 CentOS 7。一般情况下安装 Nginx 推荐添加 EPEL (全称 Extra Package for Enterprise Linux, 即企业级 Linux 附加包)安装源，然后通过 CentOS 包管理器 yum 安装。这种方式的安装简单且方便升级。因为安装模块需要重行编译 Nginx 安装包，所以这里介绍通过二进制包来安装，步骤如下。
1. 下最新的 Nginx 稳定版本二进制包，以版本 1.10.3　为例, 命令如下：
```
wget http://nginx.org/download/nginx-1.10.3.tar.g
```

2. 解压缩安装包，命令如下：
```
tar zxvf nginx-1.10.3.tar.gz
```

3. 克隆创酷镜像，命令如下：
```
git clone git@github.com:alibaba/nginx-http-concat.git
```

4. 切换到 nginx 目录，命令如下：
```
cd nginx-1.10.3
```

5. 在构建参数中添加模块并构建，代码如下：
```
./configure --add-module=/path/to/nginx-http-concat
make && make install
```

6. 查找 Nginx 地址，命令如下：
```
// 找到 Nginx 执行文件的地址，可以配置命令行环境，或者通过路径调用
whereis nginx
// 查找到 Nginx 的默认配置文件地址，如果出现异常，默认在/etc/nginx/conf.d/default.conf
nginx -t
```

通过 cat 命令读取配置文件，可以看到 root 地址，对应网站根目录。

7. 启动 Nginx, 命令如下：
```
// 推荐使用systemctl来启动，也可以通过 nginx 命令来启动
sudo systemctl start nginx
//　开机自启动，可选
sudo systemctl enable nginx
```

## 7.2 配置 Nginx 和 Combo
　　nginx-http-concat Nginx 适用于 Nginx 的文件合并模块，可以将多个对静态资源的 HTTP 请求合并成为一个，进而减少 HTTP 请求数。假设原来需要 a.js, b.js, c.js 三个文件，产生三次请求，通过该模块可以在一个请求中完成，实例地址如下:
```
//使用??作标识表示合并文件
http://example.com/static/??a.js,b.js,c.js
```

需要修改 Nginx 的配置，文件配置如下：
```
location /static/ {
    # nginx-http-concat 主开关
    concat on
    # 最大合并文件数
    # concat_max_files 10；
    # 只允许通内省文件合并
    # concat_unique on
    # 允许合并的文件类型，多个以逗号分隔。如: application/x-javascript, text/css
    # concat_types text/html
}
```
