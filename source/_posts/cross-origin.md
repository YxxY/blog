---
title: Cross-Origin 
date: 2017-12-25 20:50:38
categories:
- Common-sense
- Cross-origin
tags: 
- Cross-Origin
- CORS
- JSONP
---
前端跨域(cross-origin)是个应该掌握的基本常识，网上很多资料都有讲，全面总结下加深印象
<!--more-->
# 背景
跨域资源共享：[Cross origin resource sharing(CORS)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)

浏览器的安全策略-同源策略: [same origin policy)](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

总结来说

一个源的定义：

如果`协议`，`端口`（如果指定了一个）和`域名`对于两个页面是相同的，则两个页面具有相同的`源`

请求`非同源`资源的请求叫做`跨域 HTTP 请求`

出于安全原因，浏览器限制`从脚本内发起的`跨域HTTP请求

具体如何限制呢？
- 请求成功，拦截响应（大部分是这条）
- 拦截请求（例如部分浏览器不允许HTTPS跨域访问HTTP）

但是！跨越请求资源的需求不可或缺，所以需要想办法绕过限制

# 示例
## AJAX中的同源限制
实例示范跨域请求资源被拦截

打开浏览器，在console中输入以下代码
```js
var xhr = new XMLHttpRequest()
xhr.open('GET', 'https://baidu.com')
xhr.onload = function(){
    console.log(xhr.responseText)
}
xhr.send()
```
可以看到请求是成功的，但是没有响应，会看到一串错误信息, 提示没有允许跨域请求头

原因就是XHR遵循了浏览器的同源策略
## 天然跨域
然而上述场景很容易联想到另外的情况：表单提交
```html
<html>
<form action="https://www.baidu.com" method="get"> 
   <input type="submit" value="提交"> 
</form> 

</html>
```
很显然这个是可以跳转成功的，类似的还有
- `<a>`标签，GET请求， 会刷新或离开页面
- `<img>`src属性，GET请求， 加载外部图片
- `<link>`标签， GET请求，加载css等资源
- `<script>`， GET请求，加载脚本资源

这些都实现了获取非同源资源的请求，为什么没有被浏览器限制呢？

有人称之为“天然跨越”，因为浏览器认为这些访问是安全的，从脚本内发起的资源请求则不在该范围内

# 跨域方法
虽然有些html标签实现了“天然跨域”，但这远远不够，页面不总是静态的，如何让AJAX实现跨域请求资源呢，传统的三个方法如下：
## CORS
CORS 是一个 W3C 标准，它允许浏览器向跨源服务器，发出 XMLHttpRequest 请求，从而克服了 ajax 只能同源使用的限制。

用法也很好理解，`响应头部`添加字段告诉浏览器，响应符合规范，是安全合法的，从而让浏览器不拦截响应

eg:

`'Access-Control-Allow-Origin': http://127.0.0.1:3000`

字段值包括源的三个“身份值”，协议，域名，端口

当然这种设置只能是`服务器`去做，服务器有权决定自己的资源对谁开放

另外一个不太算缺点的缺点，该标准仅支持IE10 及以上

## JSONP
JSONP 全称为：JSON with padding，可用于解决老版本浏览器的跨域数据访问问题。

用法如下：

前端逻辑：
```html
// jsonp/index.html
<script>
    function jsonpCallback(data) {
        alert('获得 X 数据:' + data.x);
    }
</script>
<script src="http://localhost:3000?callback=jsonpCallback"></script>
```
后端用nodejs来模拟
```js
// jsonp/server.js
const url = require('url');
	
require('http').createServer((req, res) => {
	const data = {
		x: 10
	};
	// 拿到回调函数名
	const callback = url.parse(req.url, true).query.callback;
	console.log(callback);
	res.writeHead(200);
	res.end(`${callback}(${JSON.stringify(data)})`);

}).listen(3000);
```

原理：

    利用<script>标签src属性的天然跨越，向后端传递参数，响应返回为函数调用

这个函数是前端事先准备的（jsonpCallback），函数参数由后端提供，后端响应是`jsonpCallback(data)`, 即调用函数，实现动态交互

优点：
- 不受浏览器同源策略影响
- 兼容性好

缺点：
- 只支持GET请求，原因是JSONP的实现是“借鸡下蛋”，`<script>`的src属性只支持GET请求方式获取资源
- 无法捕获连接异常

## 服务端代理
这种方式依然是对同源策略的一种妥协，原理是在同源域名下使用代理转发请求

eg:

`'/proxy?url=http://baidu.com'`

后端取得参数后自行访问目标资源，得到结果后再返回给浏览器
