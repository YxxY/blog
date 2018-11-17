---
title: Node.js events模块(上)
date: 2017-06-18 20:11:38
categories:
- JavaScript
- Events
tags: 
- Node.js
- events
- EventEmitter
---

node.js 基于异步事件驱动(Event-driven)，这无疑也是一种十分优秀的编程思想，这里总结下其中与之相关的一个核心模块——events模块。
本文主要总结使用的API，具体可参考[官网API](https://nodejs.org/api/events.html)
<!--more-->

## 快速使用
`events`是内置模块，但非全局变量，使用时需显示引用
```js
const EventEmitter = require('events');
```
`events` 提供的接口是一个类，所有调用时赋值变量需大写，且需要自行实例化。
官方给出的例子如下：
```js
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

myEmitter.on('event', () => {
  console.log('an event occurred!');
});
myEmitter.emit('event');
```
这里使用了ES6的语法，创建一个`MyEmitter`类继承自初始类`EventEmitter`,方便自定义修改。

## 事件对象
myEmitter对象，可以当做一个事件触发器，发送一个特定事件。也可以当成一个事件监听器，侦听某一特定事件，并调用对应的事件响应函数。
打印一下对象,可以看到有四个属性
```js
{
     domain: null, /*ignore*/
     _events: { event: [Function] }, /*注册事件的监听函数*/
     _eventsCount: 1,  /*监听到的事件个数*/
     _maxListeners: undefined /*最大监听数，默认是10个，超出会报错*/
}
```

### 传参
当作为触发器时，需要给处理函数传递参数。
如下例，其中特别说明回调函数中的this指向监听器对象，这里指的是myEmitter。如果使用箭头函数就不会这样了，这和箭头函数的特性有关，我之前已经总结过了。
```js
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

myEmitter.on('event', function(a, b){
  console.log(a, b, this);
  /*Prints:
       a b MyEmitter {
         domain: null,
         _events: { event: [Function] },
         _eventsCount: 1,
         _maxListeners: undefined }*/
});
myEmitter.emit('event', 'a', 'b');
```
### 同步还是异步
一个监听器上可以绑定多个响应函数，监听器调用响应函数默认是按照事件注册的顺序`同步调用`的，这样可以保证原有的一些业务逻辑，先注册先调用。但如果多个响应函数之间没有任何联系，就可以使用`setImmediate()`或`process.nextTick()`切换到异步流程，如下例：
```js
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

myEmitter.on('event', (a, b) => {
  setImmediate(() => {
    console.log('this happens asynchronously');
  });
});
myEmitter.on('event', (a, b) => {
  process.nextTick(() => {
    console.log('this happens asynchronously, too');
  });
});
myEmitter.emit('event');
```
### 监听器响应函数可仅运行一次
当使用 `eventEmitter.on()`方法监听事件时，事件发生时，每次都会调用响应函数。但换成`eventEmitter.once()`就会仅处理一次。

### 错误事件
当一个事件实例对象当发生错误时，会发出`error`事件，这将被Node.js当做特殊事件处理。
如果该对象未对错误事件注册任何处理函数，该错误就会上抛，打印堆栈信息，进而导致Node.js`进程退出`。所以说后果还是很严重的。
为防止进程崩溃，必须注册至少一个监听函数。如下：
```js
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

process.on('uncaughtException', (err) => {
  console.error('whoops! there was an error');
});

/*myEmitter.on('error', (err) => {
  console.error('whoops! there was an error');
});*/ adviced!

myEmitter.emit('error', new Error('whoops!'));
// Prints: whoops! there was an error
```
## API
截止到现在，官房API总共包括了2个事件，15个方法(一个即将废弃，不做说明)
### Event：'newListener'
当有新的监听函数注册时触发，回调函数有两个参数，监听函数名和监听函数
```js
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

myEmitter.on('newListener', (event, listener) => {
    console.log(event, listener.name)
    /* print test testCb
       and this callback will invoke before testCb*/
});

myEmitter.on('test', testCb);

function testCb(){
    console.log('A');
}

myEmitter.emit('test');
```
### Event：'removeListener'
类似'newListener', 当某个事件的监听函数被移除时触发。
例如使用`myEmitter.once()`触发一次之后,监听函数就会被移除
```js
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

myEmitter.on('newListener', (event, listener) => {
    console.log(`add ${event}, ${listener.name}`)
});

myEmitter.on('removeListener', (event, listener) => {
    console.log(`remove ${event}, ${listener.name}`)
});

myEmitter.once('test', testCb);

function testCb(){
    console.log('A')
}

myEmitter.emit('test');
```
### EventEmitter.defaultMaxListeners
类属性。最大默认监听数，缺省值为10，实例对象可以使用`emitter.setMaxListeners(n)`方法修改自身的最大监听数，如果直接修改`EventEmitter.defaultMaxListeners`则会影响所有实例。

### emitter.addListener(eventName, listener)
`emitter.on(eventName, listener)`的简写，没有区别

### emitter.emit(eventName[, ...args])
同步按注册顺序调用监听`eventName`的回调函数。
返回值为Boolean，如果该事件有监听器返回true，否则返回false

### emitter.eventNames()
返回一个数组，包含该监听器上注册的所有事件名

### emitter.getMaxListeners()
返回当前最大监听数，该值可以由`emitter.setMaxListeners(n) `设置，默认等于`EventEmitter.defaultMaxListeners`

### emitter.listenerCount(eventName)
返回事件名为`eventName`的监听数

### emitter.listeners(eventName)
返回一个事件名为`eventName`的所有监听函数的数组的拷贝

### emitter.on(eventName, listener)
监听事件`eventName`,触发时调用回调函数`listener`

### emitter.once(eventName, listener)
监听事件`eventName`,首次触发时调用回调函数`listener`，然后移除监听该事件

### emitter.prependListener(eventName, listener)
将监听函数`listener`插入到事件`eventName`触发时处理函数队列的最前面。
返回值为`emitter`。
```js
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

myEmitter.on('newListener', (event, listener) => {
    console.log(`add ${event}, ${listener.name}`)
});

myEmitter.once('test', testCb);

function testCb(){
    console.log('A')
}

var that = myEmitter.prependListener('test', function(){
  console.log('someone connected!'); /* print before 'A' */
  console.log(this === that); /*true*/
});

myEmitter.emit('test');
```

### emitter.prependOnceListener(eventName, listener)
同上，区别不再赘述

### emitter.removeAllListeners([eventName])
移除某事件的所有监听
返回emitter

### emitter.removeListener(eventName, listener)
移除一个具体事件的监听，必须给出事件名和回调函数名。
因此，回调函数需要引用名。
值得注意的时，在触发某一事件后，注册在该事件上的回调函数就会按顺序执行一圈，在运行过程中再去删除监听器是不会生效的。因为……懒得写了,自己去官网看吧
返回emitter

### emitter.setMaxListeners(n)
设置默认最大监听数
返回emitter

## 总结
总结完这块官网所有API，基本掌握了如何使用，但这还远远不够。
events作为Node.js核心模块之一，采用纯javascript编写，暴露的API简洁实用，源码修改至今加上copyright也不过五百多行,非常适合精读。
本次API的总结主要为下次解读这一块的源码做铺垫。详情可参考[events.js](https://github.com/nodejs/node/blob/master/lib/events.js)