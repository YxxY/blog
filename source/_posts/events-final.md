---
title: Node.js events模块(下)
date: 2017-09-01 20:15:27
categories:
- JavaScript
- Events
tags: 
- Node.js
- events
- EventEmitter
---
又一次来看这部分源码了，感觉每次依然有收获，再做一次总结，强行分成上中下三篇，内心毫无波澜，甚至有点想笑
<!--more-->

# 关于使用
- 直接引入 EventEmitter 类，new 一个对象来使用
- 从EventEmitter继承

一般第二种居多，继承的方法很多，简单举例
```js
var obj = Object.create(EventEmitter.prototype)
```
现在obj对象就具备事件驱动的功能了
```js
obj.on('test', ()=>{
    console.log('receive event test')
})

obj.emit('test')
```
# 事件驱动的原理
这是一个典型的面向对象的例子，一个对象具备监听和发送消息的方法，所以关键就是这些方法的实现了。
该对象的两个重要属性：
- `_events`：初始化是一个空对象，用来存储事件名及对应的回调函数。key为事件名，value为回调函数，如果回调函数超过一个，则转为一个数组，用来保证顺序。
- `_maxListeners`: 注册的最大监听数，默认值为10，这是继承的一个类属性，通过Object.defineProperty()定义。

每次监听事件，即向obj._events里增加一对值，触发事件时即根据事件名调用相应回调函数，如果回调是数组类型则按顺序调用内部函数，该顺序也与注册函数时的顺序保持了一致。

# 实现技巧
了解原理后发现其实很简单，自己也可以写一个类似的包，但这个包几经修改到当前额版本，很多细节是值得学习和借鉴的。
自己动手用ES6的语法写了个简易版的该模块
```js
class Events{
    constructor(){
        this._events = Object.create(null);
        this._eventsCount = 0;
        this._maxListeners = 10;
    }

    getMaxListeners(){
        return this._maxListeners
    }

    setMaxListeners(n){
        if(typeof n !== 'number' || n < 0 || isNaN(n))
            throw new TypeError ("\"n\" argument must be a positive number")
        this._maxListeners = n
        return this
    }

    on(type, listener){
        if(typeof listener !== 'function')
            throw new TypeError ("\"n\" argument must be a positive number")
        var events= this._events
        var existing = events[type]
        if(existing){
            if(typeof existing === 'function')
                existing = [existing, listener]
            else
                existing.push(listener)
        }else
            existing = listener
        
        this._eventsCount++
        if(!existing.warn){
            var m = this.getMaxListeners()
            if(m && m < existing.length){
                existing.warned = true;
                const w = new Error('Possible EventEmitter memory leak detected. ' +
                                    `${existing.length} ${String(type)} listeners ` +
                                    'added. Use emitter.setMaxListeners() to ' +
                                    'increase limit');
                w.name = 'MaxListenersExceededWarning';
                w.emitter = this;
                w.type = type;
                w.count = existing.length;
                process.emitWarning(w);
            }
        }
        events[type] = existing
    }

    emit(type){
        var events = this._events
        var handler = events[type]
        if(!handler)
            return false
        var len = arguments.length;
        var args = new Array(len - 1)
        for (var i = 1; i < len; i++)
          args[i - 1] = arguments[i];
        var isFn = typeof handler === 'function'
        if(isFn){
            handler.apply(this, args)
        }else{
            handler.forEach(v=>{
                v.apply(this, args)
            })
        }
        return true
    }
}
```
这是个十分简陋的实现，对比源码就能发现很多细节非常值得推敲，这里重点说一个。

## Once的封装
源码实现了只触发一次回调函数，并几乎没有修改原来的接口。
```js
EventEmitter.prototype.once = function once(type, listener) {
  if (typeof listener !== 'function')
    throw new TypeError('"listener" argument must be a function');
  this.on(type, _onceWrap(this, type, listener));
  return this;
};
```
完整复用了`on`的接口, _onceWrap的实现无疑值得学习。
```js
function _onceWrap(target, type, listener) {
  var state = { fired: false, wrapFn: undefined, target, type, listener };
  var wrapped = onceWrapper.bind(state);
  wrapped.listener = listener;
  state.wrapFn = wrapped;
  return wrapped;
}

function onceWrapper() {
  if (!this.fired) {
    this.target.removeListener(this.type, this.wrapFn);
    this.fired = true;
    switch (arguments.length) {
      case 0:
        return this.listener.call(this.target);
      case 1:
        return this.listener.call(this.target, arguments[0]);
      case 2:
        return this.listener.call(this.target, arguments[0], arguments[1]);
      case 3:
        return this.listener.call(this.target, arguments[0], arguments[1],
                                  arguments[2]);
      default:
        const args = new Array(arguments.length);
        for (var i = 0; i < args.length; ++i)
          args[i] = arguments[i];
        this.listener.apply(this.target, args);
    }
  }
}
```
_onceWrap返回的是绑定了this的onceWrapper函数，存在`this._events`中。state对象做了两件事，提供this以及保存地址。
保存地址方面尤为巧妙，保存了自身、调用对象以及需要传递的参数。
其中`onceWrapper`函数中，
```js
this.target.removeListener(this.type, this.wrapFn);
this.fired = true;
```
这两行只保留任何一行也不会影响结果。双重保障，即使删除listener未成功也能防止其被二次调用。

再总结一点，从源码可知，调用回调函数时都是绑定了this，指向调用对象本身的。因此，如果想要在回调函数中使用调用者对象，
则不适合使用箭头函数。箭头函数绑定父上下文，给回调函数使用apply或call绑定this无任何效果。

# 总结
对于events模块应该不会再有总结了，毕竟题目都用完了。感觉自己还是基本掌握原理及使用的。
脑子里留下多少算多少，实在不行就再看一遍，不然写在这里岂不白写了。



