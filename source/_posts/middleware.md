---
title: middleware 
date: 2018-01-08 20:50:38
categories:
- JavaScript
tags: 
- middleware
- Node.js
---

nodejs的经典web框架中很多均实现了中间件系统，设计合理，使用也十分方便，本文主要总结这些精简api背后的源码实现
<!--more-->
## 背景
nodejs的web框架应该都是基于原生httpServer实现的，先回到原点
```js
var http = require('http')

http.createServer(function(req, res){
    res.end('hello world')
}).listen(3000)
```
nodejs提供了足够简单的api，几行代码就能构建一个http服务器

运行后访问`http://127.0.0.1:3000/`, 应该就能看到熟悉的字符

## 中间件概念

在这里请允许我对中间件做一个望文生义的解释

    一个请求可能需要被多次处理，将处理流程模块化，每一个处理环节即为中间件
    处理结束后可以选择终止处理返回结果，也可以将控制权转交给下一个模块

这是我根据框架的功能实现总结的概念，不够全面，但足够清晰

## connect的实现
选择connect来说明是因为它足够精简
```js
var connect = require('connect');
var http = require('http');
var log = console.log.bind(console)

var app = connect();

app.use(function(req, res, next){
  log('middleware 1')
  next()
});

app.use(function(req, res, next){
  log('middleware 2')
  next()
});

// respond to all requests
app.use(function(req, res){
    log('middleware 3')
  res.end('Hello from Connect!\n');
});

//create node.js http server and listen on port
http.createServer(app).listen(3000);
```
connect的api也十分简单
- 通过`app.use`注册中间件
- `next`函数做流程控制，处理结束后显示的调用next函数进入下一个中间件

运行后访问`http://127.0.0.1:3000/`, 应该可以看到控制台的依次打印
- middleware 1
- middleware 2
- middleware 3

## connect源码分析
```js
module.exports = createServer;

var proto = {};
var EventEmitter = require('events').EventEmitter;
var merge = require('utils-merge');

function createServer() {
  function app(req, res, next){ app.handle(req, res, next); }
  merge(app, proto);
  merge(app, EventEmitter.prototype);
  app.route = '/';
  app.stack = [];
  return app;
}
```

整个connect模块对外接口为一个函数，调用后仍然返回一个app函数

先看注册流程
### 中间件注册
`app.use`实现了中间件的挂载, api有如下几类
- `app.use(path, func)`, 第一个参数为注册路径，如果省略则默认为`/`, 将匹配所有请求，第二个参数为一个函数，即对该请求做相应处理
- `app.use(path， _app)`, `_app`是和`app`一样，均为`connect()`返回的函数对象，是第一类api的特殊情况，相当于挂载子应用
- `app.use(path, httpServer)`, 同上，也是特殊处理的情况，接受一个httpServer对象，处理时调用它的第一个请求处理函数

源码如下：
```js
proto.use = function use(route, fn) {
  var handle = fn;
  var path = route;

  // default route to '/'
  if (typeof route !== 'string') {
    handle = route;
    path = '/';
  }

  // wrap sub-apps
  if (typeof handle.handle === 'function') {
    var server = handle;
    server.route = path;
    handle = function (req, res, next) {
      server.handle(req, res, next);
    };
  }

  // wrap vanilla http.Servers
  if (handle instanceof http.Server) {
    handle = handle.listeners('request')[0];
  }

  // strip trailing slash
  if (path[path.length - 1] === '/') {
    path = path.slice(0, -1);
  }

  // add the middleware
  debug('use %s %s', path || '/', handle.name || 'anonymous');
  this.stack.push({ route: path, handle: handle });

  return this;
};
```
了解了api再来看源码，会清楚很多，做的事情即：
- 中间件注册时需两个参数
- 第一个是路径参数，如果省略则默认为`/`，匹配所有路径。
- 第二个参数为处理参数（有普通函数，子应用，httpServer对象，共三种不同的表现形式）
- 随后路径和处理函数会被打包成一个对象存储在数组`app.stack`中

完了再看处理流程
### 中间件调用

app函数接受三个参数，对比原生api，多了一个next参数，在不了解的情况下，我们可以先做个如下实验
```js
var http = require('http')

http.createServer(function(req, res, next){
    res.end('goodbye world')
    console.log(next)
}).listen(3000)
```
运行后访问，控制台打印为`undefined`, 可以肯定这个参数并非一开始存在的，而是后续封装的

通过`http.createServer(app).listen(3000);`可知，当有请求消息时，处理流程为

        app(req, res) => app.handle(req, res, next) 
        此时的next为undefined

继续看app.handle函数
```js
proto.handle = function handle(req, res, out) {
  var index = 0;
  var protohost = getProtohost(req.url) || '';
  var removed = '';
  var slashAdded = false;
  var stack = this.stack;

  // final function handler
  var done = out || finalhandler(req, res, {
    env: env,
    onerror: logerror
  });

  // store the original URL
  req.originalUrl = req.originalUrl || req.url;

  function next(err) {
    if (slashAdded) {
      req.url = req.url.substr(1);
      slashAdded = false;
    }

    if (removed.length !== 0) {
      req.url = protohost + removed + req.url.substr(protohost.length);
      removed = '';
    }

    // next callback
    var layer = stack[index++];

    // all done
    if (!layer) {
      defer(done, err);
      return;
    }

    // route data
    var path = parseUrl(req).pathname || '/';
    var route = layer.route;

    // skip this layer if the route doesn't match
    if (path.toLowerCase().substr(0, route.length) !== route.toLowerCase()) {
      return next(err);
    }

    // skip if route match does not border "/", ".", or end
    var c = path[route.length];
    if (c !== undefined && '/' !== c && '.' !== c) {
      return next(err);
    }

    // trim off the part of the url that matches the route
    if (route.length !== 0 && route !== '/') {
      removed = route;
      req.url = protohost + req.url.substr(protohost.length + removed.length);

      // ensure leading slash
      if (!protohost && req.url[0] !== '/') {
        req.url = '/' + req.url;
        slashAdded = true;
      }
    }

    // call the layer handle
    call(layer.handle, route, err, req, res, next);
  }

  next();
};
```
虽然是个大函数，但做的事情很简单
- 接受三个参数
- 变量`done`被设置为`finalhandler`函数的返回值
- 定义`next`函数
- 调用`next`函数

next函数内部的操作：
- 按顺序从`app.stack`中取出一个对象
- 如果不匹配，递归调用`next`函数，相当于取下一个对象
- 如果遍历完都没有匹配的，调用`done`函数
- 匹配成功，调用`call(layer.handle, route, err, req, res, next)`

层层包装，到了`call`函数这里就是最后一层了
```js
function call(handle, route, err, req, res, next) {
  var arity = handle.length;
  var error = err;
  var hasError = Boolean(err);

  debug('%s %s : %s', handle.name || '<anonymous>', route, req.originalUrl);

  try {
    if (hasError && arity === 4) {
      // error-handling middleware
      handle(err, req, res, next);
      return;
    } else if (!hasError && arity < 4) {
      // request-handling middleware
      handle(req, res, next);
      return;
    }
  } catch (e) {
    // replace the error
    error = e;
  }

  // continue
  next(error);
}
```
- 通过handle函数（注册时的处理函数）形参个数来做不同处理，4个参数则是错误处理，3个参数则是请求处理
- 不管是成功还是失败，next均作为参数传入
- 如果处理错误继续调用next(err)

至此，前文的流程可以继续走了

如果匹配到注册路径的中间件，则

        app(req, res) => app.handle(req, res, undefined) => handle(req, res, next)

如果中间抛出错误，且注册了错误处理函数（相比正常的错误函数多了一个参数，形如func(err, req, res, next))

        app(req, res) => app.handle(req, res, undefined) => handle(err,req, res, next)

如果一个都没匹配到

        app(req, res) => app.handle(req, res, undefined) => finalhandler(req, res, {env: env,onerror: logerror})(err)


next函数是通过闭包的方式实现，手动决定控制权的转移，也能控制错误的冒泡处理

至此，connect的原理基本都解析完毕了，源码就一个文件，注释加空行不到三百行但实现的功能却非常巧妙，如果你已经非常熟悉了中间件的api，那么这份源码非常值得一看

## 总结
总结即是看完源码，写完上面一大坨后闭上眼看还剩下啥

- 中间件的概念，自己编
- 中间件的实现，注册路径和处理函数，存在一个数组中，后续根据请求路径遍历数组，因此注册顺序很重要
- 控制权转让，手动调用next()即进入下一个中间件，如果出错也可以传递错误参数next(err), 交给后续错误处理中间件集中处理
- 中间件函数类型
    
        普通处理函数，处理完调下一个，此时必须有三个参数，eg：func(req, res, next)
        响应返回函数，最后一道程序，处理完返回结果，此时不需要next函数，所以注册时两个或三个参数均可
        错误处理函数，原则上应该放在最后注册，且参数必须为四个, eg: func(err, req, res, next), 当然可以注册多个错误处理函数，不同的错误依旧可通过next(err)向后传递
- next 函数的实现，闭包，内部保留有存储中间件的数组以及传入的初始索引，递归调用或当做参数传递调用，可以达到遍历数组的效果

嗯……似乎没啥剩下了，就这么多