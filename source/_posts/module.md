---
title: Node.js 模块
date: 2017-06-25 21:15:38
categories:
- JavaScript
- Module
tags: 
- Node.js
- module
- require
---

Node.js 中的模块(module)是一开始就接触到的，但也许是因为它的API已足够简单，实际却从未深究过。另外在新的ES6标准里，之前所用的`require`、`exports`将被`import`、`export`取代成为新的标准。新旧更替如果两个都不懂就太尴尬了，我决定查下资料，稍微总结对比下新旧的不同，争取从不懂变成略懂。
<!--more-->

# API
`require()` 参数为字符串，加载模块,返回被加载模块`module.exports`的内容
`require.cache` 返回已加载模块的对象，key为模块绝对路径名，value为module对象
`require.resolve()` 参数为字符串，运用require()的机制去查找模块并不加载，返回模块绝对路径名
`require.main` 源码解释`require.main = process.mainModule` 指向主模块，当不存在入口文件时返回 undefined

# require-exports
Node应用由模块组成，采用CommonJS模块规范。根据这个标准，每一个文件就是一个模块。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。除非它主动对外提供接口。每个模块内部都有一个Module对象，代表当前模块，由模块内一个全局变量`module`表示。
自定义`module.exports`表示需要导出的接口。`exports`等同于module.exports的引用。
`require`的用法：参数为字符串
- 如果参数字符串以`'/'`开头，表示绝对路径
- 如果参数字符串以`'./'`开头，表示相当路径
- 如果参数字符串不含`'/'`或`'./'`，则表示加载的是一个默认提供的核心模块，或者一个位于各级node_modules目录的已安装模块（全局安装或局部安装）
- 如果参数字符串表示的是一个目录，则会自动查看该目录的package.json文件，然后加载main字段指定的入口文件。如果package.json文件没有main字段，或者根本就没有package.json文件，则会加载该目录下的index.js文件或index.node文件
- 如果指定的模块文件没有发现，Node会尝试为文件名添加`.js`、`.json`、`.node`后，再去搜索。.js件会以文本格式的JavaScript脚本文件解析，.json文件会以JSON格式的文本文件解析，.node文件会以编译后的二进制文件解析。

require加载模块即加载被加载模块的`module.exports`部分内容，且存入缓存，再次加载时会优先从缓存加载，相当于原模块的copy，原模块的变化不会对已加载内容产生影响。同理，如果对加载内容进行修改，相当于修改内存数据，会影响到后续使用。

# import-export
新的ES6标准，变化主要在于
- import 必须作用在顶层作用域, 不能在局部作用域动态加载，目前为止还不能完全取代require
- import命令加载其他模块，`.js`后缀不可省略
- ES6 模块不会缓存运行结果，而是动态地去被加载的模块取值，并且变量总是绑定其所在的模块
- ES6 输入的模块变量，只是一个“符号连接”，所以这个变量是只读的，对它进行重新赋值会报错

相同的是
- 同一个模块如果加载多次，将只执行一次

# require-module
`module`代表当前模块，是一个对象，它的父类是
```js
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  if (parent && parent.children) {
    parent.children.push(this);
  }

  this.filename = null;
  this.loaded = false;
  this.children = [];
}
module.exports = Module;
```
require 和 module 的关系是
```js
/* Loads a module at the given file path. Returns that module's
   `exports` property. */
Module.prototype.require = function(path) {
  assert(path, 'missing path');
  assert(util.isString(path), 'path must be a string');
  return Module._load(path, this);
};
```

# 参考来源
[Node.js API](https://nodejs.org/api/globals.html#globals_require)
[CommonJS规范](http://javascript.ruanyifeng.com/nodejs/module.html#toc6)
[require() 源码解读](http://www.ruanyifeng.com/blog/2015/05/require.html)
[ECMAScript6 Module的语法](http://es6.ruanyifeng.com/#docs/module)







