---
title: express框架学习(上)
date: 2017-09-4 20:46:55
categories:
- JavaScript
- Express
tags: 
- express
- Node.js
---
项目中选用的web框架是express，总结下经验
<!--more-->
# express是做什么的

从官网所得描述是基于node.js的一个简洁灵活的web框架，性能上没有对node进行二次抽象，也就是说express的性能的上限就是node本身的性能。
那就先从node本身说起。

网络编程这块，node代表性的有两大模块，`net`和`http`，（`https`基于`http`先不提）

使用`net`模块写一个简单的`hello world`服务器如下
```js
const net = require('net');

var server = new net.Server()

server.listen(3329)

server.on('connection', (socket)=>{
    console.log(`receive a connection from ${socket.address().address}`)
    socket.on('data', (data)=>{
        console.log(`receive:\n${data}`)
        socket.write("HTTP/1.1 200 OK \n\nHello World!\r\n")
        socket.destroy()
    })
})
```
node运行后，可以在浏览器访问`http://localhost:3329/`,没什么意外的话就能看到熟悉的hello world。这里运用的是socket编程。
浏览器每一次的http请求和服务器建立一次socket连接。从收到的内容打印看，收到的确实是http格式的请求，服务器的回复也是符合http的响应格式的。
响应头加两个换行符后接响应体，少一个换行符就看不到hello world了。
但这个服务器太简陋了，它分不清不同的请求，所有都回复相同的内容。原因当然是没有解析请求的数据了，需要解析打印出来的`data`内容。

于是就有了下面这段代码。

```js
const http = require('http');

var server = http.createServer((req, res)=>{
    console.log(`receive a request: ${req.method} ${req.url} HTTP${req.httpVersion}`);
    res.write("Hello, World!")
    res.end()
});

server.listen(3329)

```
和上面功能相同，但仿佛更加简洁了。可以直接得到请求的url，以及回响应时不用关心响应的格式，直接写我们最关注的响应体就可以了。这些方便正是http模块相对于net模块，做了更多的封装，完成了解析请求数据的工作，并提供十分便捷的API。
到这个版本就可以针对不同的访问请求回复不同的响应了，如下：
```js
const http = require('http');

var server = http.createServer((req, res)=>{
    console.log(`receive a request: ${req.method} ${req.url} HTTP${req.httpVersion}`);
    if(req.method === 'GET' && req.url === '/'){
        res.write("Welcome Home!")
    }else{
        res.write("Hello, World!")
    }
    res.end()
});

server.listen(3329)
```
当访问``http://localhost:3329/hello`时返回`Hello, World!`，访问`http://localhost:3329/`, 返回`Welcome Home!`。
当然这是一个非常粗糙的实现，如果页面较多时代码将变得极其难看和很难维护。

下面该`express`登场了。
```js
var http = require('http')
var express = require('express')

var app = express();
var server = http.createServer(app)
server.listen(3329)

app.get('/', (request, response)=>{
    response.send('Welcome Home!')
})

app.get('/hello', (request, response)=>{
    response.send('Hello World!')
})
```
同样实现了之前的需求，但是写法简洁清晰不少，最重要的是方便维护。除此之外，引入的`express`、`app`、`request`、`response`，都提供了极其丰富且实用的API，具体可参考[官网API](http://expressjs.com/en/4x/api.html)。
然而以上是比较老的写法，最新版本的express写法更加简单，http模块整个都被封装了。
```js
var express = require('express')
var app = express()

app.get('/', function (req, res) {
  res.send('Hello World')
})

app.listen(3329)
```
总的来说就是，node.js自带web server，express为该server打造了更加健壮的基础。


