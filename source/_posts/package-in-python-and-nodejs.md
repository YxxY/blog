---
title: 包机制：Python vs Node.js
date: 2018-1-31 20:45:27
categories:
- Python
tags: 
- python
- node.js
- module
---
专注在nodejs上有一段时间了，回头看看python发现忘得差不多了，还是得多做做总结加强记忆啊。
先从包机制入手，对比来看下python和nodejs的异同
<!--more-->

## python package
### 导出模块
python将一个文件或一个文件夹视为一个模块

如果是文件件，需要显示的创建`__init__.py`来标明

模块内定义的所有变量都可以被导出

python中的变量不强制使用前声明，默认为局部变量，但使用前必须被定义或赋值

全局变量需要用`global`声明

### 导出模块
python2.6以后及python3 默认支持的是绝对引用，也推荐这种方式

通过`import`关键字引入模块, 模块名不需要带`.py`后缀
```python
import pkg
from pkg import foo
from pkg.moduleA import foo
from pkg.moduleA import foo as bar
from pkg import *
```

python解释器会自动在`sys.path`下去搜索`pkg`包

windows下可以定义`PYTHONPATH`使得路径永久生效

`python -m`的方案可解决一些包引入错误，原理即使将`当前目录`加到`sys.path`中，
相当于提供了顶级目录。

引入模块的时候会执行内部代码，一些不想在引入时执行的内容一般处理如下：
```python
# pkg.py
def test():
    print '123'

print __name__  #pkg
if __name__ == '__main__':
    test()
```
当作为模块引入时`__name__` 为包名，直接执行时为`__main__`, 以此来做区分

引入一个文件夹包，相当于引入它目录下的`__init__.py`, 一般为空文件

如果在里面引入子模块，则后续可以用`.`的形式链式调用
```python
"""
pkg
├── __init__.py
└── main.py
test.py
"""
# main.py
a = 2

# __init__.py 为空时，从test.py 引入变量a
# test.py
from pkg.main import a
print a

# __init__.py
import main

# 可以在__init__.py中提前引入模块，方便后续调用
# test.py
import pkg
print pkg.main.a

```

模块的`__all__`变量可以定制对外暴露的对象，不在其中的内容不能被外部引用
```python
"""
pkg
├── test.py
└── main.py
"""
# main.py
__all__ = ['a']

a = 2
b = 3
# test.py
from main import *
print a
print b  
```
执行 `test.py`,会得到错误`# NameError: name 'b' is not defined`

注释掉`__all__`那行就不会报错了

## node package
### 导出模块
node中一个文件就是一个模块

如果是文件夹，则目录下的`index.js`为默认导出文件

导出的内容和python不同，`必须`显性指定到`module.exports`上，该值默认为`{}`

ES6的语法，则是利用`export`关键字去实现导出
```js
exports = module.exports = {}
//ES6 
export var a = 1;
export {a}
export {a as b}
export default a;
```

### 导入模块
node中包导入方式：
- 传统的`require`函数
- ES6的`import`函数

先看require函数的用法
#### 模块名导入
直接使用`不带路径`的模块名一般为内置包或安装的第三方包
```js
var fs = require('fs')
var express = require('express')
```
#### 相对路径导入
导入模块的后缀可省
```js
var fs = require('./index')
```
会寻找当前目录下的`index.js`,如果不存在则认为`index`是一个文件夹，
继续寻找`./index/index.js`

#### 绝对路径导入
没什么可说的，参数数为模块的绝对路径

#### import函数的用法
```js
import _ from 'lodash'
import _, { each, each as forEach } from 'lodash';
import { a } from './someModule';
```

## 总结
python 和 nodejs 均以单个文件为`module`, 同时也支持文件夹的包裹，细分又有很多不同

二者包的路径查找也有很大区别，python需要保证包在系统路径下，添加PYTHON_PATH可以永久添加

nodejs是在当前目录下查找，同时也支持全局安装包
