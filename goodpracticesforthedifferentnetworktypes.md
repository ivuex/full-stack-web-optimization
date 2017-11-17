# goodPracticesForTheDifferentNetworkTypes

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
