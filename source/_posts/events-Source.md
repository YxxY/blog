---
title: Node.js events模块(中)
date: 2017-06-19 20:11:38
categories:
- JavaScript
- Events
tags: 
- Node.js
- events
- EventEmitter
---

总结完`events`模块后会发现，这一块的API十分的简洁高效，对它的实现十分好奇。因为这块源码比较简短，代码编写十分规范，于是我选择更进一步，尝试下学习下源码[events.js](https://github.com/nodejs/node/blob/master/lib/events.js)。
<!--more-->

## 模块的编写
通常我们都会自己写模块，但对比下这篇专业的，对自己以后写模块有些许启发
```js
function EventEmitter() {
  EventEmitter.init.call(this);
}
module.exports = EventEmitter;
```
这是最开头的部分，`module.exports`写在文件开头，导出的部分一目了然，我个人比较习惯这种方式，不喜欢手动拉到最后去看这块信息。
另外，将要导出的函数模块化，没有把所有的内容都写到函数里面，解耦带来的好处不用赘述，也是非常合理的。
函数的call方法忘记的话参考[Function.prototype.call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

## 初始化
EventEmitter类实例化就是调用`EventEmitter.init()`函数
```js
EventEmitter.prototype._events = undefined;
EventEmitter.prototype._maxListeners = undefined;

EventEmitter.init = function() {
  this.domain = null;
  if (EventEmitter.usingDomains) {
    /* if there is an active domain, then attach to it. */
    domain = domain || require('domain');
    if (domain.active && !(this instanceof domain.Domain)) {
      this.domain = domain.active;
    }
  }

  if (!this._events || this._events === Object.getPrototypeOf(this)._events) {
    this._events = Object.create(null);
    this._eventsCount = 0;
  }

  this._maxListeners = this._maxListeners || undefined;
};
```
domain 模块即将被废弃，可以不关心，从第二个if开始看。
这个判断的第二部分`this._events === Object.getPrototypeOf(this)._events)`存在的必要性我至今没看明白，但可以理解下字面意思。对象的`_events`属性和构造函数prototype的`_events`属性的比较，这里就是`undefined === undefined`的情况。
而且这里需注意`Object.getPrototypeOf(this)`和`this.prototype`不同这点，类似`obj.__proto__`和`obj.prototype`之间的关系。

值得学习的第二个点就是`this._events = Object.create(null)`。
之前的版本是`this._events = {}`。
所以`Object.create(null)`和`{}`之间肯定是有区别的，前者对象属性更少，创建速度更快。可参考[Creating Objects With A Null Prototype In Node.js](https://www.bennadel.com/blog/2797-creating-objects-with-a-null-prototype-in-node-js.htm)

初始化过程很简单，就是初始化3个属性：`_events`置成一个空对象，用来后续存储监听器回调函数。`_eventsCount`属性置0，后期计数。`_maxListeners`,重写类属性`defaultMaxListeners`。
实例对象的其他属性和方法都从`EventEmitter.prototype`继承。

## 定义属性
EventEmitter类定义`defaultMaxListeners`时，用的如下实现方式：
```js
/* By default EventEmitters will print a warning if more than 10 listeners are
   added to it. This is a useful default which helps finding memory leaks. */
var defaultMaxListeners = 10;

Object.defineProperty(EventEmitter, 'defaultMaxListeners', {
  enumerable: true,
  get: function() {
    return defaultMaxListeners;
  },
  set: function(arg) {
    /* force global console to be compiled.
       see https://github.com/nodejs/node/issues/4467 */
    console;
    /* check whether the input is a positive number (whose value is zero or
       greater and not a NaN). */
    if (typeof arg !== 'number' || arg < 0 || arg !== arg)
      throw new TypeError('"defaultMaxListeners" must be a positive number');
    defaultMaxListeners = arg;
  }
});
```
与通常使用的`obj.property`定义方式不同，这里使用了[Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)。
应用在这里的配置使得`defaultMaxListeners`属性值不可被删除，可以赋值，且赋值前有检查。

## 最大监听数的重写
```js
/* Obviously not all Emitters should be limited to 10. This function allows
   that to be increased. Set to zero for unlimited. */
EventEmitter.prototype.setMaxListeners = function setMaxListeners(n) {
  if (typeof n !== 'number' || n < 0 || isNaN(n))
    throw new TypeError('"n" argument must be a positive number');
  this._maxListeners = n;
  return this;
};

function $getMaxListeners(that) {
  if (that._maxListeners === undefined)
    return EventEmitter.defaultMaxListeners;
  return that._maxListeners;
}

EventEmitter.prototype.getMaxListeners = function getMaxListeners() {
  return $getMaxListeners(this);
};
```
这里采取是如果对象属性存在就返回对象属性，否则返回类属性。
另外发现一个没什么意义的点，对非负数且不能为正无穷的判断这里用的是`typeof n !== 'number' || n < 0 || isNaN(n)`,在之前定义`defaultMaxListeners`属性时也有个相同的需求，写的是`typeof arg !== 'number' || arg < 0 || arg !== arg`，从这点也大概能推测这个库就不是一个人写的。其实还可以发现，最大监听数为浮点数也是可以的。
其次，这里之所以抽象出`$getMaxListeners`函数，而不是全部逻辑写到`getMaxListeners`里，是因为在`addListener`的接口里也需要判断当前最大监听数，提取出来可以减少重复代码。

## 触发器传参的优化
在事件触发时，触发器传递的参数数量是任意的，引用源码注释
```commment
    These standalone emit* functions are used to optimize calling of event
    handlers for fast cases because emit() itself often has a variable number of
    arguments and can be deoptimized because of that. These functions always have
    the same number of arguments and thus do not get deoptimized, so the code
    inside them can execute faster.
```
即固定参数函数调用比不定参数的调用效率更高，因此`emitter.emit()`函数传参个数在`三个以内`效率是很高的。
这里再以函数为三个的情况举例说明
```js
len = arguments.length;
switch (len) {
  /* fast cases */
  case 1:
    emitNone(handler, isFn, this);
    break;
  case 2:
    emitOne(handler, isFn, this, arguments[1]);
    break;
  case 3:
    emitTwo(handler, isFn, this, arguments[1], arguments[2]);
    break;
  case 4:
    emitThree(handler, isFn, this, arguments[1], arguments[2], arguments[3]);
    break;
  /* slower */
  default:
    args = new Array(len - 1);
    for (i = 1; i < len; i++)
      args[i - 1] = arguments[i];
    emitMany(handler, isFn, this, args);
}
```

```js
function emitThree(handler, isFn, self, arg1, arg2, arg3) {
  if (isFn)
    handler.call(self, arg1, arg2, arg3);
  else {
    var len = handler.length;
    var listeners = arrayClone(handler, len);
    for (var i = 0; i < len; ++i)
      listeners[i].call(self, arg1, arg2, arg3);
  }
}
```
当同一事件有多个监听者时，监听函数会被按顺序放进一个数组，这里做的操作时先复制一份这个数组，然后再依次调用内部函数。
也就是说，在运行期间原数组的内部的增删不会对执行产生任何影响。
也因为这个操作避免了一些死循环事件的发生。

## 添加监听器
因为`emitter.on`和`emitter.prependListener`存在重复的逻辑，所以这段实现代码也单独抽象出`_addListener`函数。
```js
function _addListener(target, type, listener, prepend) {
  var m;
  var events;
  var existing;

  if (typeof listener !== 'function')
    throw new TypeError('"listener" argument must be a function');

  events = target._events;
  if (!events) {/* 还没看出这里if存在的意义 */
    events = target._events = Object.create(null);
    target._eventsCount = 0;
  } else {
    /* To avoid recursion in the case that type === "newListener"! Before
       adding it to the listeners, first emit "newListener". */
    /* 即第一次侦听'newListener'事件不会触发'newListener' 回调函数 */
    if (events.newListener) {
      target.emit('newListener', type,
                  listener.listener ? listener.listener : listener);

      /* Re-assign `events` because a newListener handler could have caused the
         this._events to be assigned to a new object */
      events = target._events;
    }
    existing = events[type];
  }

  if (!existing) {
    /* Optimize the case of one listener. Don't need the extra array object. */
    existing = events[type] = listener;
    ++target._eventsCount;/* 新的事件名才会增加计数 */
  } else {
    if (typeof existing === 'function') {
	/* 这一段的排序写的很有意思 */
      /* Adding the second element, need to change to array. */
      existing = events[type] = prepend ? [listener, existing] :
                                          [existing, listener]
    } else {
      /* If we've already got an array, just append. */
      if (prepend) {
        existing.unshift(listener);
      } else {
        existing.push(listener);
      }
    }

    /* Check for listener leak */
    if (!existing.warned) {
      m = $getMaxListeners(target);
      if (m && m > 0 && existing.length > m) {
        existing.warned = true;
        const w = new Error('Possible EventEmitter memory leak detected. ' +
                            `${existing.length} ${String(type)} listeners ` +
                            'added. Use emitter.setMaxListeners() to ' +
                            'increase limit');
        w.name = 'MaxListenersExceededWarning';
        w.emitter = target;
        w.type = type;
        w.count = existing.length;
        process.emitWarning(w);
      }
    }
  }

  return target;
}
```
这一块主要是写逻辑判断，当然代码也很精炼，特别说三点：
 - `newListener`事件注册位置很重要，必须要在其他事件之前才能出现期待的效果
 - `newListener`事件的回调参数分别为新增的监听事件名和`listener.listener ? listener.listener : listener`
 - 在事件触发之前添加的监听器的回调函数才会被调用

## 移除监听器
如果事件只有一个监听函数，即`_events`对象只含有一个属性，这是采用`delete`方式删除。
若事件有多个监听函数，则对应是从数组中移除特定位置元素，如果是最开头的位置，当然是`list.shift()`,是末尾就用`list.pop()`,那么中间位置呢？
第一个想到的当然是`list.splice(index, 1)`,但这里用了更加效率的方法。
```js
/* About 1.5x faster than the two-arg version of Array#splice(). */
function spliceOne(list, index) {
  for (var i = index, k = i + 1, n = list.length; k < n; i += 1, k += 1)
    list[i] = list[k];
  list.pop();
}
```

## once的实现
源码采用三个函数实现了once的功能
```js
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

function _onceWrap(target, type, listener) {
  var state = { fired: false, wrapFn: undefined, target, type, listener };
  var wrapped = onceWrapper.bind(state);
  wrapped.listener = listener;
  state.wrapFn = wrapped;
  return wrapped;
}

EventEmitter.prototype.once = function once(type, listener) {
  if (typeof listener !== 'function')
    throw new TypeError('"listener" argument must be a function');
  this.on(type, _onceWrap(this, type, listener));
  return this;
};
```

而之前的版本是
```js
function _onceWrap(target, type, listener) {
  var fired = false;
  function g() {
    target.removeListener(type, g);
    if (!fired) {
      fired = true;
      listener.apply(target, arguments);
    }
  }
  g.listener = listener;
  return g;
}
```
相比较，原来的版本似乎更简洁，使用闭包, `fired`和`g()`的组合，保证`listener`只会调用一次。拆除两个函数，还是不定参数转固定参数以获取效率提升。
拆开只会闭包就用不上了，新版本采用了`bind`的写法，也是可以借鉴的。

## 总结
我觉得对我自己现在而言，我所接触到的代码上限就是我能写出的代码上限。毫无疑问这次的源代码学习拔高了我平时接触到的代码的上限，收获也颇多，总结为以下几点：
- 编写一个类模块时可以参考events的设计：代码解耦合，使用prototype、this等关键字，使得代码精简，结构清晰
- 尽可能的提取公共逻辑，减少代码重复
- 特殊技巧的使用可以参考，比如定义对象特殊属性、对象属性覆盖类属性的实现
- 性能提升，events模块主要涉及到空对象创建、不定参数的函数调用转为固定参数调用、删除数组中间部分元素等方法，可以试着化为己用。






