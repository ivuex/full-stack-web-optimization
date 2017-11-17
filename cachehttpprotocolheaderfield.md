# cacheHTTPProtocolHeaderField

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