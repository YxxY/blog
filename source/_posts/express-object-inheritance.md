---
title: express源码中的对象继承
date: 2018-01-10 21:50:38
categories:
- JavaScript
- Express
tags: 
- express
- Object-inheritance
---

nodejs web框架`express`源码中, 多处封装涉及到了对象继承，单独拿出来总结下它们的特点，参考学习
<!--more-->
##  对象继承方式
简单阅读源码所看到的对象继承方式有三种
- merge-descriptors

```js
var EventEmitter = require('events').EventEmitter;
var mixin = require('merge-descriptors');
//app对象从EventEmitter.prototype对象继承
mixin(app, EventEmitter.prototype, false);
```
- Object.create()

```js
//app.request对象从req对象继承，并定义app属性
app.request = Object.create(req, {
    app: { configurable: true, enumerable: true, writable: true, value: app }
  })
```
- Object.setPrototypeOf()

```js
var setPrototypeOf = require('setprototypeof')
//将router对象的原型对象设置为proto
setPrototypeOf(router, proto)
```

## 源码分析
这应该是最常用的的三种继承形式了

### 属性拷贝
第一种为属性值拷贝，没什么技巧就是把要继承的属性全部拷贝到自己身上, 可以选择是否覆盖同名属性

源码的实现如下
```js
//merge-descriptors
module.exports = merge

var hasOwnProperty = Object.prototype.hasOwnProperty

/**
 * Merge the property descriptors of `src` into `dest`
 *
 * @param {object} dest Object to add descriptors to
 * @param {object} src Object to clone descriptors from
 * @param {boolean} [redefine=true] Redefine `dest` properties with `src` properties
 * @returns {object} Reference to dest
 * @public
 */

function merge(dest, src, redefine) {
  if (!dest) {
    throw new TypeError('argument dest is required')
  }

  if (!src) {
    throw new TypeError('argument src is required')
  }

  if (redefine === undefined) {
    // Default to true
    redefine = true
  }

  Object.getOwnPropertyNames(src).forEach(function forEachOwnPropertyName(name) {
    if (!redefine && hasOwnProperty.call(dest, name)) {
      // Skip desriptor
      return
    }

    // Copy descriptor
    var descriptor = Object.getOwnPropertyDescriptor(src, name)
    Object.defineProperty(dest, name, descriptor)
  })

  return dest
}
```

拷贝继承，继承时会耗费更多时间和空间，但后续访问时理论上效率更高

常驻于内存中的对象更加适合这种继承方式, 一次继承多次使用

后两种可归为一类，均是从原型对象继承
### 原型继承
原型继承，是js中常见的的继承方式，这里有两种操作方法

- `Object.create(proto[, propertiesObject])`

        返回在指定原型对象上添加新属性后的对象
        即除了继承之外，还可以自定义属性

- `setprototypeof`

```js
//setprototypeof
module.exports = Object.setPrototypeOf || ({__proto__:[]} instanceof Array ? setProtoOf : mixinProperties);

function setProtoOf(obj, proto) {
	obj.__proto__ = proto;
	return obj;
}

function mixinProperties(obj, proto) {
	for (var prop in proto) {
		if (!obj.hasOwnProperty(prop)) {
			obj[prop] = proto[prop];
		}
	}
	return obj;
}
```
一般来说调用的是原生接口`Object.setPrototypeOf`

同时也提供了两种兼容方案
- `obj.__proto__ = proto;` 
- 属性值拷贝

## 总结
自己写代码或者写库程序不可避免会涉及到继承封装，express提供了很好的最佳实践
- 对于常驻对象推荐使用`属性拷贝`，用空间换时间
- 原型继承有`Object.create()`和`Object.setPrototypeOf()`等方法，分别有各自适合的场景，更多了解可以参照MDN的说明
- 不想自己造轮子，express的这俩久经考验的轮子也可以直接拿来使用，`merge-descriptors`和`setPrototypeOf`, 源码也十分简洁