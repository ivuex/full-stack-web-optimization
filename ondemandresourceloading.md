# onDemandResourceLoading

## 5. 资源按需加载
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
