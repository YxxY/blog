---
title: 浏览器事件 ready vs load
date: 2018-04-12 22:50:38
categories:
- front-end
tags: 
- document
- ready
- jquery
---

前端面试一个经常会被问的问题，document ready 和 load 事件有什么区别  
类似的问题还有`$(document).ready`函数和`window.onload`函数有什么区别
<!--more-->

简单来说，`ready`事件是浏览器DOM加载完成的标志，此时文档内的DOM树已加载完成，但是不包括图片和第三方资源  
`load`事件则是所有资源加载完毕，所有网络请求时结束时触发  
一图流解释如下：
![dom-loader](/images/dom-load.png)  
蓝色部分`DomContentLoaded` 对应DOM加载完成，触发ready 事件  
红色`Load`的时间会久一点，因为还要加载图片和异步请求，完成时触发load 时间

## 应用
`document ready`一般可借助`jQuery`实现应用，常见的三种语法是：
- `$(document).ready(callback)`
- `$().ready(callback)`
- `$(callback)`

`load`事件可以用`window.onload = callback`来监听

根据上面的简单总结，很显然$(document).ready(callback) 的回调函数要比 window.onload **提前调用**  

具体用谁，得根据自己的需求来定，即定义函数调用的时间

另外常说的一点就是，就是ready事件被jquery封装后可以多次调用，即可以这么写
```js
$(document).ready(callback1)
$(document).ready(callback2)
```
他们会按注册顺序执行，但是`window.onload` 不行，如果你这么写
```js
window.onload = callback1
window.onload = callback2
```
那么只有后面的回调函数会覆盖前面的执行


## 思考实践
- `document ready`是文档同步加载完成（仅指页面内`<html`>标签所包含的元素）的标志，它能监听异步的文档渲染么？
- 浏览器拿到服务端的html是如何完成解析的

以下主要探讨这两个问题
### 文档的异步渲染
先说第一个结论，`jquery ready`只是文档同步加载完成的标志，它并不能监测到`异步的文档渲染`。

验证如下：

延迟3秒向页面中添加一个`p`元素，并监听`document.ready`事件。

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="./js/jquery.min.js"></script>
    <script>
      $(document).ready(function () {
          alert('document is ready')
      });
    </script>
  </head>
  <body>
    <div>
      this is a div
    </div>
    <button>button</button>
    <script>
      setTimeout(() => {
        $('body').append("<p>You can't see me</p>")
      }, 3000);
    </script>
  </body>
</html>
```

由于`alert`有中断浏览器渲染的作用，所以`alert`出现时，观察页面元素即可得出结论。

结果是:
- 页面立即显示`alert`的内容
- 出现`alert`时，`<div>`和`<button>`元素已经存在，说明确实是加载完成才触发的`ready`事件。
- 点击alert的确认，三秒后添加的`<p>`元素出现，说明`alert`的中断确实起作用，而且异步文档渲染不在`ready`事件监控范围内。

### 文档渲染流程

根据[Web API](https://developer.mozilla.org/en-US/docs/Web/API/Document/readyState)
浏览器文档的加载状态 `Document.readyState` 分三个阶段
- `loading`
  - 加载中
- `interactive`
  - 文档已经完成加载并解析完成，但例如图片，样式文件、子窗口等子资源仍在加载中，对应`ready`事件，同步文档已经渲染完成，可以获取DOM
- `complete `
  - 所有子资源均加载完成，触发`load`事件。如果此时注册`window.onload`回调函数，则会被调用

eg：
```js
switch (document.readyState) {
  case "loading":
    // The document is still loading.
    break;
  case "interactive":
    // The document has finished loading.
    // We can now access the DOM elements.
    var span = document.createElement("span");
    span.textContent = "A <span> element.";
    document.body.appendChild(span);
    break;
  case "complete":
    // The page is fully loaded.
    let CSS_rule = document.styleSheets[0].cssRules[0].cssText;
    console.log(`The first CSS rule is: ${CSS_rule }`);
    break;
}
```

总结来说，我们日常说的页面加载完成，基本指的是当前页面内的元素以**同步**的方式加载完成。

## More Details

前两部分的总结，基本清楚了html文档的整体加载过程，知道了大体分三个阶段，每个阶段都有哪些标志，但还不够具体。

查阅资料得知页面的渲染机制如下流程图所示：

![html_parsing](/images/html_parsing.png)


以webkit内核的渲染流程举例来说：

![webkit-render-flow](/images/webkit-render-flow.png)

浏览器拿到html的解析由渲染引擎来做，这部分是不同于网络资源加载时的多线程，而是单线程的
- 从`<html>`节点开始分析，从上到下
- 遇到`<script>`, 对于外链的JavaScript文件，需要先加载该文件内容，再进行解析，然后立即执行。
  这整个过程都会`阻塞文档解析`，直到脚本执行完才会继续解析文档节点。
  所以一般`<script>`标签都建议放在最后的`</body>`元素之前，HTML5提供`defer`和`async`两个属性支持延迟和异步加载JavaScript文件
- 针对上面说的脚本阻塞文档解析，主流浏览器如Chrome和FireFox等都有一些优化，
  比如在执行脚本时，开启另外的线程(一般限制在2-6个)解析剩余的文档，
  找出并加载其他的待下载外部资源（不改变主线程的DOM树，仅优化加载外部资源）。
- 对样式的处理也会阻塞文档解析，加载外部样式时后续js和文档解析都不会继续进行
- 在js脚本中请求样式信息，如果样式尚未加载或解析，将会得到`错误信息`
- 为了更友好的用户体验，浏览器会尽可能快的展现内容，而不会等到文档所有内容到达才开始解析和构建/布局渲染树，
  而是每次处理一部分，并展现在屏幕上，这也是为什么我们经常可以看到页面加载的时候内容是从上到下一点一点展现的。


## 总结

现在在让我回答

> 如何确定一个页面完全加载完成？

这个问题不能想的太细，一般来说监听`load`事件即可

> 如何确定异步请求的结束？

只能通过异步请求自身的消息监听，外部无法获取它的状态变化

> `document ready`事件和`window load`事件分别是什么动作完成的标志
    
前者是`当前文档内`（异步DOM操作不包含在内）的DOM树构建完成的标志，可以进行DOM操作了，但是还没有完成渲染  
后者是整个页面加载并渲染完成的标志

> js的下载和执行顺序能保证么？

js的下载顺序不能保证，因为网络部分是多线程下载的  
执行顺序是可以保证的，会按照页面`<script>`标签出现的顺序执行内部同步代码（异步操作加入事件循环），因为渲染引擎是单线程的

> 页面的解析渲染流程？

从html头开始，一边解析一边渲染，主线程会被外部资源的加载以及js的执行阻塞，因此资源放置的位置以及顺序是有影响的

## 参考
- [Web API](https://developer.mozilla.org/en-US/docs/Web/API/Document/readyState)
- 《WebKit技术内幕》









