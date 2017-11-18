# yahooPerformanceOptimizationUnderTheEffectiveLaw

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

+ **减少DOM元素次数**
　　DOM被认为天生就慢，通常需要将经常访问的DOM对象缓存，避免过多的使用JavaScript修改页面布局。

+ **避免使用iframe**
　　即使引入页面内容为空，iframe也需要消耗时间下载, 会阻止页面加载，并且缺少语义。

+ **优化图像**
　　设计师提供的图片大多数都存在压缩空间，或者可以GIF格式转换为PNG格式。

+ **优化 CSS Sprites**
　　CSS Sprites中，图片水平排列较之垂直排列体积更小，可以将颜色接近的组合在一起。不要在Sprites中留有较大的空隙。空隙虽然不会增加文件大小, 但是对于用户需要跟多的内存来把图片解压为像素地图。

+ **不要在HTML中缩放图片**
　　在HTML中设置的图片长宽小于实际下载图片长宽,很显然实际加载的图片比需要的图片体积更大。

+ 减少Cookie体积

  - Cookie信息通过HTTP文件头在服务器和浏览器之间进行传递,因此保持Cookie尽可能小,可以减少用户的数据传输时间。

　- 以上列了雅虎法则中前端优化最常用的几条,更多的内容可以查阅雅虎开发者官方网站。通常一个网页从发起服务器请求到实现对最终用户的展现，在客户端所花费的时间大约会占到总时耗的80%，因此客户端的性能优化显得尤为重要。这也是成为优秀前端工程师必须要掌握的基础知识。