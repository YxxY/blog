---
title: RobotFramework Library API
date: 2017-7-22 21:45:27
categories:
- Python
- RobotFramework
tags: 
- RobotFramework
---
RF本身是用Python写的，它的扩展库也可以用Python来写。
如果是跑在Jython上，也是可以用java来扩展。
纯python的扩展具备通用性，本文也主要总结基于python的RF扩展库的API，RF版本为截止到当前最新的V3.0.2
<!--more-->
## API 分类
RF有三种不同的库API:
- 静态API, 最常用的。提供一个包或者一个类，里面的公有方法名会被映射为`关键字名`
- 动态API, 动态的生成和执行关键字，除了运行时生成的特点，其他和静态API相同
- 混合API, 即动态的引用静态API创建的关键字

本文仅总结静态API,其它两类差不太多，只是有一些炫酷的用法而已

## 创编Library库
Library的名字导入后会和包名或者类名保持一致

eg：现在引入一个Python库 MyLibrary(一个文件，MyLibrary.py)，
它将创建一个名字也为MyLibrary的库

### 编写类
通常一个包里包含的是一个类，如果类名和包名相同，RF允许在导入时省略类名

如果类名和包名不一致，则Library名需要补全导入的类名

举例说明
```python
# Test.py
class Test(object):
    '''testing library'''
    def __init__(self):
        pass
    def foo(self, str1, str2):
        return str1 + str2
```
现在引入`Test.py`, 则库名为`Test`
```robot
Library     Test
```
此时`foo`关键字可以直接使用，接受两个参数返回拼接后的内容

但如果改变类名为`Foo`, 在执行就会报错`找不到关键字foo`

这个时候两个解决方案，将类名改成和包名一致，或者更改导入库如下：
```robot
Library     Test.Foo
```
其实，之前的导入库相当于`Test.Test`的别名
### 编写函数和变量
除了类之外也可以创建变量和函数, 包命名空间里的方法也会变成关键字

包括导入的库名
```python
# test.py
from threading import current_thread

bar = 233
def test():
    print 'hello world'
```
分别引入
```robot
Library           test
Variables         test.py
```
`current_thread`和`test`会成为关键字

`${bar}`会成为可使用的变量

函数、变量不能和类同时导出

原因我猜大概是导出类时其实是精确指定类名了，自然不会包括其他内容

如果不想current_thread被导出，有两种方法
- 导入时，写成`import threading`,使用时写成`threading.current_thread()`
- 通过包属性`__all__`限定导出的方法和变量

### 库参数
如果是导入的是一个类，自然是可以接受参数的，这些参数会被传递给构造器创建类实例
```robot
*** Settings ***
Library    MyLibrary     10.0.0.1    8080
Library    AnotherLib    ${VAR}
```
参数会保证顺序和个数传递，这个很好理解

需要注意的是RF里默认的参数类型是字符串，所以传递的数字会被转换

### 库引入方式
一般Library为一个文件或第三方库，引入的方式有两种：
- 在测试套里直接引入`Library`
- 在测试套里引入`Resource`，在Resource文件里引入Library

第二种方式多用于多处引入或批量引入

### 库作用域
当引入的库是各种类时，它们是可以有一个内部状态的，内部状态会在初始化和关键字执行时被改变，同时也会影响关键字的行为。

说简单点就是，关键字执行其实是执行类实例的方法，那其实就是这个类实例是每个用例一个实例还是大家共有一个实例的问题

默认行为是每个用例一个实例，互不影响

但这个行为也是可控的，通过一个类属性`ROBOT_LIBRARY_SCOPR`

该属性有三个值
- `TEST_CASE`, 默认值
- `TEST_SUIT`, 一个测试套共用一个实例
- `GLOBAL`, 全局单例

eg：
```python
class ExampleLibrary:

    ROBOT_LIBRARY_SCOPE = 'TEST SUITE'

    def __init__(self):
        self._counter = 0

    def count(self):
        self._counter += 1
        print self._counter

    def clear_counter(self):
        self._counter = 0
```

### 库版本
同库作用域，通过类属性`ROBOT_LIBRARY_VERSION`定义

如果不存在会尝试从`__version__`属性读取

### 文档格式化
同上，通过类型`ROBOT_LIBRARY_DOC_FORMAT`指定文档格式，后续可以使用`Libdoc`工具生成对应格式的文档

属性值大小写不敏感

可能的值有`ROBOT`(default),`HTML`,`TEXT`

也可以使用第三方工具`docutil`,属性值为`reST`

### 创建静态关键字
前面说过，如果是静态API，RF会自动的将引入库里的公有方法转化为关键字

这里面仍有一些细节需要说明
- 以下划线开头的方法会被忽略
- 关键字的解析大小写不敏感
- 关键字中的下划线和空格会被忽略

eg：
```python
def _helper_method(self, arg):
        return arg.upper()

def hello(name):
    print "Hello, %s!" % name

def do_nothing():
    pass
```
引用时`_helper_method`不能被使用

`hello`方法对应的关键字可以是`hello`, `Hello`, `h e l l o`

`do_nothing`或`doNothing`的关键字名都可以是`Do Nothing`

#### 自定义关键字名
关键名也可以按照自己的意愿命名

实现上最先想到的应该就是装饰器了，实际也是如此

通过`robot.api.deco.keyword`在方法上设置一个`robot_name`属性实现

```python
from robot.api.deco import keyword

@keyword(u'登录')
def login(username, password):
    # ...
```
使用时, 直接使用`登录`即可，原来`login`不再映射为关键字
```robot
*** Test Cases ***
My Test
    登录   ${username}    ${password}
```
从RF v3.0.2开始，通过这种方式，即使是以下划线开头的私有方法也能转换为可用关键字

##### 关键字嵌套参数
关键字还可以接受嵌套参数，示例如下：
```python
from robot.api.deco import keyword

@keyword('Add ${quantity:\d+} Copies Of ${item} To Cart')
def add_copies_to_cart(quantity, item):
    # ...
```
使用
```robot
*** Test Cases ***
My Test
    Add 7 Copies Of Coffee To Cart
```

### 打印信息
异常信息会被打印，除此之外，向标准输出流(stdout/stderr)写入的信息也会被写入log文件，同时也还可定义不同的log level

#### 打印级别
默认是`INFO`, 除此之外还有`TRACE`, `DEBUG`,`WARN`,`ERROR`,`HTML`

`WARN`,`ERROR`级别的信息会被自动写入到控制台

如果是`HTML`, 则支持所有html语法的展现，eg：`<b>foo</b>`，但需要慎用`<table>`便签，支持不是很好

#### 打印到控制台
```python
import sys

def my_keyword(arg):
   sys.__stdout__.write('Got arg %s\n' % arg)

# The final option is using the public logging API:
from robot.api import logger

def log_to_console(arg):
   logger.console('Got arg %s' % arg)

def log_to_console_and_log_file(arg):
   logger.info('Got arg %s' % arg, also_console=True)

def log_with_html_format():
    logger.info('<i>This</i> is a boring example', html=True)
```
直接使用`print`函数也行
```python
print 'Hello from a library.'
print '*WARN* Warning from a library.'
print '*ERROR* Something unexpected happen that may indicate a problem in the test.'
print '*INFO* Hello again!'
print 'This will be part of the previous message.'
print '*INFO* This is a new message.'
print '*INFO* This is <b>normal text</b>.'
print '*HTML* This is <b>bold</b>.'
print '*HTML* <a href="http://robotframework.org">Robot Framework</a>'
```
结果如下：
```text

16:18:42.123	INFO	Hello from a library.
16:18:42.123	WARN	Warning from a library.
16:18:42.123	ERROR	Something unexpected happen that may indicate a problem in the test.
16:18:42.123	INFO	Hello again!
This will be part of the previous message.
16:18:42.123	INFO	This is a new message.
16:18:42.123	INFO	This is <b>normal text</b>.
16:18:42.123	INFO	This is bold.
16:18:42.123	INFO	Robot Framework
```

## 总结
RF框架包含的内容很多，了解一些基本使用足以支撑应用，更详细的用法参考[官方文档](http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html)

本文主要参考官方文档一部分[Extending Robot Framework](http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#id704)
