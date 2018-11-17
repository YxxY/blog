---
title: XMLHttpRequest
date: 2017-12-29 20:50:38
categories:
- Common-sense
tags: 
- XMLHttpRequest
- AJAX
---
据说前端培训班第一课就会讲`XMLHttpRequest`和`AJAX`。
不说别的，至少能得出结论，这是个很基础，很重要的概念。
我没去过培训班，在此总结一下，如果被问到这个问题，我会怎么回答。
<!--more-->
## 背景
- XMLHttpRequest (XHR) 是干嘛用的

        使用xhr对象发送http请求和服务器通信
        你可以通过一个url传输和获取服务端的数据
        因为以前多用来传递xml格式的数据，所以名字里有个xml
        但xhr可以发送和接收多种类型的数据资源（现在更多的是JSON、html、text files）
- XHR 是什么

        是一个API
        是一个类
        是一个构造函数
- 为什么要用XHR
        
        支持页面局部更新
        不用整体刷新页面，不会打断用户操作，用户体验好

- 哪些地方用到了XHR    

        XMLHttpRequest 在 AJAX 中被大量使用
- 所以AJAX 又是啥

        全称Asynchronous JavaScript And XML
        它使用XHR对象和服务器通信
        它是XHR的一个封装
        特点是Asynchronous（异步）
### 总述

XMLHttpRequest 是一个底层封装好的API, 通过它可以很容易的取回一个 URL 上的资源数据。尽管名字里有 XML，但 XMLHttpRequest 支持的数据类型并不局限于 XML。而且除了 HTTP ，它还支持 file 和 ftp 协议。

AJAX 则是对XMLHttpRequest的进一步封装，让使用更加方便

了解XMLHttpRequest，对AJAX的理解会更加深入

本文着重讲XMLHttpRequest

## XMLHttpRequest
想通过javascript发送一个http请求，我们需要一个具有必备功能的对象去做这件事，这个对象就是`XMLHttpRequest`对象

这个概念最初由微软设计，在IE中作为一个"ActiveX"对象使用，被称作"XMLHTTP",随后 Mozilla、Apple 和 Google跟进, 
搞出了一个"XMLHttpRequest"对象，功能上和ActiveX对象差不多。如今，该对象已经被 W3C组织标准化。

以下代码可以很好的说明这段历史
```js
// Old compatibility code, no longer needed.
if (window.XMLHttpRequest) { // Mozilla, Safari, IE7+ ...
    httpRequest = new XMLHttpRequest();
} else if (window.ActiveXObject) { // IE 6 and older
    httpRequest = new ActiveXObject("Microsoft.XMLHTTP");
}
```
现代浏览器直接调用构造函数`new XMLHttpRequest()`即可创建一个新的xhr对象
_ _ _

对象有了，怎么发送请求呢，当然调用对象方法
```js
httpRequest.open('GET', 'http://www.example.org/some.file', true);
httpRequest.send();
```
### xhr.open(requestMethod, url, isAsync)
- 第一个参数是http请求方法名，安装http标准应该全部为大写字母
- 发送请求的url地址，出于安全性考虑，采用同源策略
- 可选参数，是否异步（js继续执行，未收到服务器响应前用户可继续与页面交互），默认为true

### xhr.send()
- 发送请求参数
- 如果是POST请求，发送表单数据时应该以一种服务器可以解析的格式，例如query字符串的形式
- 支持多种格式的数据发送，multipart/form-data, JSON, XML 等等.
- 如果是使用POST发送数据，还需要设定请求的MIME类型，即定义请求头部

下例表示请求里的参数类型为query字符串
```js
httpRequest.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
httpRequest.send("name=value&anothername="+encodeURIComponent(myVar)+"&so=on")
```
_ _ _

发送请求后，需要接受响应。

在这个阶段，需要告诉xhr对象调用哪个函数去处理响应。

通过设置`onreadystatechange`属性值为一个函数，会在请求状态变化时调用

### xhr.onreadystatechange 
- 属性值为一个函数引用
- 函数无参数
- 相当于回调函数，处理服务器的响应

```js
httpRequest.onreadystatechange = nameOfTheFunction;
```

处理函数如何处理响应呢，答案是根据请求状态
### xhr.readyState
- 0 (uninitialized) or (request not initialized)
- 1 (loading) or (server connection established)
- 2 (loaded) or (request received)
- 3 (interactive) or (processing request)
- 4 (complete) or (request finished and response is ready)

details:
- 0, `UNSENT`, Client has been created. open() not called yet.
- 1, `OPENED`, open() has been called.
- 2, `HEADERS_RECEIVED`, send() has been called, and headers and status are available.
- 3, `LOADING`, Downloading; responseText holds partial data.
- 4, `DONE`, The operation is complete.

因此，响应处理函数的内容可以是
```js
if (httpRequest.readyState === XMLHttpRequest.DONE) {
    // Everything is good, the response was received.
} else {
    // Not ready yet.
}
```
除此之外，还可以参考http响应的状态码，如`200 ok`来确认响应结果
### xhr.status
```js
if (httpRequest.status === 200) {
    // Perfect!
} else {
    // There was a problem with the request.
    // For example, the response may have a 404 (Not Found)
    // or 500 (Internal Server Error) response code.
}
```
### xhr.responseType
属性`responseType`决定了响应的数据类型，如果返回的数据类型不符合设定，响应值将被置成`null`

该属性只在异步请求中可用，同步请求设置该值会报错

支持的标准responseType如下：
- `""`, DOMString(default)
- `"arraybuffer"`
- `"blob"`
- `"document"`
- `"json"`
- `"text"`, DOMString

### xhr.response
返回对应`responseType`中的内容实体
### xhr.responseText
将响应数据以`text`类型返回一个DOMString
### xhr.responseURL
返回请求的url，如果不存在则返回空字符串
### xhr.responseXML
返回一个XML 文档解析而来的 DOM 对象，也可用来解析HTML，设置responseType为document即可
### xhr.timeout 
超时时长，单位为ms

- - -
## Tips
- 对于相同请求，浏览器会默认缓存结果，不会重复提交请求。如果要请求相同内容，记得设置请求头部`Cache-Control: no-cache`
- 请求报错时，错误会在响应处理函数内部抛出，可以使用try...catch捕获
```js
function alertContents() {
  try {
    if (httpRequest.readyState === XMLHttpRequest.DONE) {
      if (httpRequest.status === 200) {
        alert(httpRequest.responseText);
      } else {
        alert('There was a problem with the request.');
      }
    }
  }
  catch( e ) {
    alert('Caught Exception: ' + e.description);
  }
}
```
- 浏览器对xhr json格式响应数据的支持并不全面，需特殊处理，假设服务端返回的为json数据，处理可以如下
```js
function alertContents() {
  if (httpRequest.readyState === XMLHttpRequest.DONE) {
    if (httpRequest.status === 200) {
      var response = JSON.parse(httpRequest.responseText);
      alert(response.computedString);
    } else {
      alert('There was a problem with the request.');
    }
  }
}
```
## More
`XMLHttpRequest`类从`XMLHttpRequestEventTarget`和`EventTarget`继承

简单来说，一个处理http请求事件，一个可以监听事件消息，众多浏览器均以支持。

提供标准的`addEventListener()`APIs注册消息，然后设置`on*`属性为消息处理函数

因此可以看到类似以下形式的api：
- xhr.onload = reqListener
- xhr.onerror =  errorHandler
- xhr.onprogress = progressHandler

## 总结
简单过了一遍XHR，总体来看依旧是典型的对象编程，对于普通使用者无非就是熟悉api，看了XHR的api，再去看ajax的api，会感觉非常容易接受，甚至自己动手写个ajax也是不难实现的。

资料参考
[MDN XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

