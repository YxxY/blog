---
title: 记一次面试总结
date: 2018-04-12 22:50:38
categories:
- Common-sense
- Summary
tags: 
- DOM
---
最近有一个面试，本来以为挺有希望的，但还是准备不充分，说了很多即兴不过脑子的回答，可能给人留下一种很不靠谱的感觉吧。不过这都不是最重点，重点是面试中被问到了一个很有“想法”的问题，当时我没能捋清思路，回答的不是很完美。但在我回来研究了一下之后，发现面试官给的答案其实也不对……whatever，我只想总结下这次值得学习的经历。
<!--more-->

## 问题回顾
印象中记得`document ready`是网页加载完成的标志事件。一般借助`jQuery`实现应用，常见的三种语法是：
- `$(document).ready(callback)`
- `$().ready(callback)`
- `$(callback)`

作用是监听文档加载状态，在加载完成时调用回调函数做一些操作。

由于没有深入总结过，我的认知只停留在这个层次，直到我在面试中被问到这样一个问题

    如何判断一个页面完全加载完成?

- 我的回答是通过`seleniumLibrary`的库关键字`确认页面title`以及`定位一个已知页面元素`是否存在
- 这个回答被质疑，定位页面中的一个元素的存在就能确认整个页面加载完成么？
- 质疑很合理，页面中会有异步数据请求然后重新渲染DOM的操作，如何确认异步请求完成？
- 我继续回答异步请求拿AJAX举例，成功或失败时都会用回调函数，可以做一些操作当作完成标志，比如弹个窗
- 虽然回答没有大的问题，但还不是提问者心里的答案
- 我想了一会儿又补充了一个方法，在文档最后追加一个`script`，里面append一个隐藏元素，然后定位该元素的存在
- 这个答案也被否定，现在想想确实风马牛不相及，当时脑子一定短路了


最后我询问提问者能否给出好的解决方法，他说的是`document ready`。

说实话当时只感觉没get到提问者的问题点，翻译下他的问题应该是`页面加载完成有什么标志?`。

如果能get到这个点的话，我应该能反应过来。

甚至还能说一下`$(document).ready(callback)` 和 `window.onload = callback`的区别

不过回来后仍给我带来了一些思考？

- `document ready`是文档同步加载完成（仅指页面内`<html`>标签所包含的元素）的标志没问题，也能监听异步的文档渲染么？
- 浏览器拿到服务端的html是如何完成解析的

以下主要探讨这两个问题

## $(document).ready
先说结论，`jquery ready`只是文档同步加载完成的标志，它并不能监测到`异步的文档渲染`。

实例验证如下：

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
- 点击alert的确认，三秒后添加的`<p>`元素出现，说明`alert`的中断确实起作用，且异步文档渲染不在`ready`事件监控范围内。

如果觉得样例不够还可自行验证，我结合后端模拟了ajax请求，结论也是如此。

这里不再重复举例，因为原理都是一样的，异步的文档操作都是对文档的重绘或者重画，不算在当前页面加载的范围内。

`jQuery ready`只是同步文档加载完成的标志，不参与异步的监听。

## 文档渲染流程

根据[Web API](https://developer.mozilla.org/en-US/docs/Web/API/Document/readyState)
文档的加载状态 `Document.readyState` 分三个阶段
- `loading`, 仍在加载
- `interactive`， 文档已经完成加载并解析完成，但例如图片，样式文件、子窗口等子资源仍在加载中。对应`jquery ready`事件，同步文档已经渲染完成，可以获取DOM。
- `complete `，所有子资源均加载完成，触发`load`事件。如果此时注册`window.onload`回调函数，则会被调用

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

总结来说，我们日常说的页面加载完成，基本指的是当前页面内的元素以同步的方式加载完成。

加载渲染会经历三个阶段：
- loading，加载中
- interactive，加载完成（除了图片、样式、子窗口等元素），可以访问DOM元素，触发`jQuery ready`事件。
- complete, 渲染完成（不包括异步的DOM操作），样式也有了，触发`load`事件，调用注册的`window.onload`函数

## More Details

前两部分的总结，基本清楚了html文档的整体加载过程，知道了大体分三个阶段，每个阶段都有哪些标志，但还不够具体。

查阅资料得知页面的渲染机制如下流程图所示：

![html_parsing](http://opxo4bto2.bkt.clouddn.com/html_parsing.png)


以webkit内核的渲染流程举例来说：

![webkit-render-flow](http://opxo4bto2.bkt.clouddn.com/webkit-render-flow.png)

浏览器拿到html的解析由渲染引擎来做，这部分是不同于网络资源加载时的多线程，而是单线程的
- 从`<html>`节点开始分析，从上到下
- 遇到`<script>`,对于外链的JavaScript文件，需要先加载该文件内容，再进行解析，然后立即执行。这整个过程都会`阻塞文档解析`，直到脚本执行完才会继续解析文档节点。所以一般`<script>`标签都建议放在最后的`</body>`元素之前，HTML5提供`defer`和`async`两个属性支持延迟和异步加载JavaScript文件
- 针对上文说的脚本阻塞文档解析，主流浏览器如Chrome和FireFox等都有一些优化。
比如在执行脚本时，开启另外的线程(一般限制在2-6个)解析剩余的文档以找出并加载其他的待下载外部资源（不改变主线程的DOM树，仅优化加载外部资源）。
- 对样式的处理也会阻塞文档解析，加载外部样式时后续js和文档解析都不会继续进行
- 在脚本中请求样式信息，如果样式尚未加载或解析，将会得到`错误信息`
- 为了更友好的用户体验，浏览器会尽可能快的展现内容，而不会等到文档所有内容到达才开始解析和构建/布局渲染树，而是每次处理一部分，并展现在屏幕上，这也是为什么我们经常可以看到页面加载的时候内容是从上到下一点一点展现的。


## 总结

现在在让我回答

- 如何确定一个页面完全加载完成？

    - 我的回答依然会是确认页面title以及确认页面是否包含一个或一些目标元素（熟悉seleniumLibray的同学应该清楚，有两个库关键字来做这个工作，`Title Should Be`和`Element Should Contain`，当然还有一个`Wait Until Element Contains`会更合适)
    这里就算是考虑异步加载数据并重新渲染文档的情况，也可以把要加载的元素当做定位元素去确认。
    如果非要纠结异步结束的标志，这题根本没法答。因为页面上如果存在10个异步操作，怎么确认谁是最后完成的那个呢？

- 如何确定异步请求的结束？
    - 只能通过异步请求自身的消息监听，外部无法获取它的状态变化

- `document ready`事件和`window load`事件分别是什么动作完成的标志
    - 前者是`当前文档内`（异步DOM操作不包含在内）的DOM树构建完成的标志，可以进行DOM操作了，但是还没有完成渲染
    - 后者是整个页面加载并渲染完成的标志

- js的下载和执行顺序能保证么？
    - js的下载顺序不能保证，因为网络部分是多线程下载的；执行顺序是可以保证的，会按照页面`<script>`标签出现的顺序执行内部同步代码（异步操作加入事件循环），因为渲染引擎是单线程的

- 页面的解析渲染流程？
    - 从html头开始，一边解析一边渲染，主线程会被外部资源的加载以及js的执行阻塞，因此资源放置的位置以及顺序是有影响的

参考：
- [Web API](https://developer.mozilla.org/en-US/docs/Web/API/Document/readyState)
- 《WebKit技术内幕》









