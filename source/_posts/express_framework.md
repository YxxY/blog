---
title: express框架学习(中)
date: 2017-09-19 23:11:38
categories:
- JavaScript
- Express
tags: 
- express
- Node.js
---

本次总结试图说明express框架运行的原理，结论都是从阅读源码得来，但文中不会提及源代码，仅作总体认知方面进行说明。

<!--more-->
# 实例
结合一个官网实例来说明
```js
var express = require('express');
var app = express();

app.use(function(req,res, next){
  console.log(`receive: ${req.url}`)
  next()
})

app.get('/', function (req, res) {
  res.send('Hello World!');
});

var server = app.listen(3000, function () {
  var host = server.address().address;
  var port = server.address().port;

  console.log('Example app listening at http://%s:%s', host, port);
});
```
如无意外，运行以上代码即实现了一个十分简单的web版hello world，控制台还有一些打印。
总的来说，express的对外接口设计还是十分友好实用的，这里主要用到三个API
- app.use, 注册中间件
- app.METHOD，注册路由（这里用到的是具体的get方法）
- app.listen， 创建http server，监听端口

结合上例下面主要论述清楚两个问题：
1. express在创建http server前的中间件和路由的实现
2. 一条特定url的http请求是如何被express处理

# 图例
![express](http://opxo4bto2.bkt.clouddn.com/image/png/express.png)

# 中间件及路由的挂载
对应问题一。此处分为三部分：
- require得到的express是一个函数，调用后返回的app仍然是一个函数，得到app前已调用初始化函数`app.init()`完成一些默认设置。
- 继续调用`app.use`或`app.METHOD`函数，他们的效果是调用另一个函数`app.lazyrouter`,这个函数会给app对象动态增加`_router`属性。同时，根据`app.use`或`app.METHOD`函数的参数，封装成特定的layer对象，存放在`app._router.stack`属性中，该属性是一个数组，首次加载时会添加两个默认中间件，因此，无论是use还是get等方法添加的中间件还是路由，都会按顺序从第三个元素开始存放在stack数组中，因此调用函数的顺序很重要。
- 对于`app.METHOD`系列函数，`layer`对象会增加`route`属性，用于存放注册的路由路径（eg：”/test“)、路由方法（eg：“get”，以及相应的处理函数），如果是形如`app.get('/test', middleware1, ， middleware2...function(req,res){...}`, route的stack属性则按顺序存储对应逻辑转换的layer对象，并且`layer.handle`属性均为`route.dispatch`的bound函数

# 路由系统的工作原理
对应问题二。这里同样分成三部分：
- express内部封装了http模块，对应http的api`http.createServer([requestListener])`可知，即每次request请求事件都会调用requestListener函数，对应到express里就是`app`函数，`app()`里又调用了`app.handle()`函数。
- `app.handle()`函数做了两件事，一时增加一个默认的错误处理函数，如果此时不存在路由，则直接调用该错误处理函数，否则调用`app._router.handle()`函数，同时将错误处理函数作为第三个参数传入。
- `app._router.handle()`函数完成了匹配与分发的路由工作，这里通过`next`函数来控制遍历`app._router.stack`，每次调用按顺序处理一个layer，先调用`layer.match()`判断是否匹配路由，如果匹配则调用`layer.handle_request`从而调用`layer.handle()`处理函数，否则调用`layer.handle_error()`处理错误。如果继续调用`next()`函数则继续处理下一个layer, 否则视为处理结束。如果是人为的漏掉next，则会导致后续处理不会被调用。

# 总结
express 结合node自带的http模块，站在巨人的肩膀展现了封装的艺术。良好的层次结构，友好实用的api设计都是值得学习和借鉴的，后续会再尝试研究细节的实现。


