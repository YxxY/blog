---
title: Python中的元类
date: 2017-8-1 21:05:27
categories:
- Python
tags: 
- metaclass
- python
---
python中的元类在大多数场景里是用不到的，但理解元类有助于深入理解python面向对象编程中实例的创建过程，应该当做基础知识掌握。

<!--more-->
## python中的类
大多数语言里，类就是一组用来描述如何生成一个对象的代码段,python里也不例外。
```python
class ObjectCreator(object):
    pass

obj = ObjectCreator()
print(obj) # <__main__.ObjectCreator object at 0x0392E270>
```
除此之外，python的类同时也是一种对象，用`class`关键字声明类时，内存中就创建了一个对象，该对象具备在执行时创建一个实例对象的能力，但本质上仍然是一个对象，因此仍然可以进行如下操作：
- 赋值给变量
- 增加它的属性
- 作为函数参数传递

最重要的是，它是一个对象，那么就可以动态的创建它
```python
def create_class_by_name(name):
    if name == 'foo':
        class Foo(object):
            pass
        return Foo
    else
        class Bar(object):
            pass
        return Bar

myClass = create_class_by_name('foo')
```
以上代码动态的创建了一个类，这样的做法显然还不够。

回到类的定义，`类就是一组用来描述如何生成一个对象的代码段`

那有没有描述如何生成一个类的代码段呢，答案当然是存在的，因为类本身也是对象。

`一组用来描述如何生成一个类的代码段`叫做元类(metaclass)

## metaclass
简言之，实例对象是一个类的实例，类则是元类的实例
### type
`type`常用的用法是作为一个全局函数，得到对象类型，但它同时具备创建类的作用，即通过参数类型实现了函数的重载

    type(className, (base,), {property:value})

当且仅当传入三个参数时，返回创建的类
- 类名字符串，
- 需要继承的父类元祖(可以为空)
- 类属性字典。

看到这个api应该不用举例了，可以非常方便的创建和继承类

所以可以说`type`是一个类，和`int`,`str`等类一样。

同时还是一个`metaclass`，因为它的实例仍然是一个类

### 类型检查
python里的一切皆对象，对象都是从一个类创建而来，对象本身也会携带创建它的类的信息, 这也是`__class__`属性的功能
```python
>>> a = 5
>>> a.__class__
<type 'int'>
>>> a = 'hello'
>>> a.__class__
<type 'str'>
>>> def foo(): pass
...
>>> foo.__class__
<type 'function'>
>>> class Bar(object): pass
...
>>> b = Bar()
>>> b.__class__
<class '__main__.Bar'>
>>> a = 6
>>> type(a) is a.__class__
True
>>> a.__class__.__class__
<type 'type'>
```
注意，`__class__`的返回值并非字符串，而是和type作为对象类型判断的函数返回值相同，是一个对象，该对象是由`type`元类直接创建, 

如果上述最后一个例子有点绕，那再看一个例子
```python
>>> Bar = type('Bar', (), {})
>>> Bar.__class__
<type 'type'>
>>> int.__class__
<type 'type'>
>>> str.__class__
<type 'type'>
```
总结来说，type作为普通函数，可以打印出创建该对象的类的类型,如'int', 'str'等

但类自身也是一个对象，也是被创建的，创建类的类被称为元类，它的类型是`<type 'type'>`

## `__metaclass__`
`__metaclass__`可以是一个模块级别的属性，在顶层作用域会影响模块类所有类的创建

也可以作为类属性，那么它将仅影响该类的创建

原理如下：

当类声明后，在内存中创建前，会先寻找`__metaclass__`属性，
如果存在，就用它来创建类，否则使用内建的`type`来创建该类

因此，`__metaclass__`属性指向的是一个用来创建类的元类，这很符合它的命名

具体来操作下，Take One：
```python
def myMetaclass(name, base, attr):
    return type(name, base, attr)

class Foo(object):
    __metaclass__ = myMetaclass
    def __init__(self):
        pass

print(Foo.__class__)

foo = Foo()

print(foo.__class__)

# <type 'type'>
# <class '__main__.Foo'>
```
没错，我是来搞笑的……什么都没干，但是演示了一把元类的是如何生效的，以及`__metaclass__`的使用

再来写个有用的
```python
def myMetaclass(name, base, attr):
    attr['slogan'] = 'hello world'
    return type(name, base, attr)

class Foo(object):
    __metaclass__ = myMetaclass
    def __init__(self):
        pass

print(Foo.slogan)

foo = Foo()

print(foo.slogan)
# hello world
# hello world
```

操作空间很大，`name`,`base`, `attr` 都是可操作对象，
举例是给类添加了一个固定属性，还可以动态的更改
```python
def myMetaclass(name, base, attr):
    if attr.get('slogan'):
        attr['slogan'] = 'goodbye world'
    else:
        attr['slogan'] = 'hello world'
    return type(name, base, attr)

class Foo(object):
    __metaclass__ = myMetaclass
    slogan = "you can't see me"
    def __init__(self):
        pass

print(Foo.slogan) 

foo = Foo()

print(foo.slogan)

# goodbye world
# goodbye world
# 如果注释掉 slogan = "you can't see me"，打印会是两个 hello world
```
这个例子展示了动态修改类属性，根据这个实例，可以大胆推断：

类在创建前会加载所有属性和方法，存储在一个字典中，集中所有父类到一个元祖中，然后再调用元类创建实例

总的来说，自定义元类的发挥作用的流程是：
- 拦截类的创建
- 修改类, 可操作的内容有name，base，attr三个对象
- 返回修改后的类

但现在的实现并非一个类，因为`myMetaclass`是一个函数，还可以再改进

## 自定义类
上面说了`__metaclass__`实现了动态修改类，但不太算面向对象的语法。还有更简洁的方式，那就是通过定义类的`__new__`方法

### `__new__`
简单来说，一个类的实例对象是`__new__`和`__init__`方法共同创造的。
前者创建,后者完成初始化。最后返回该对象。也就是说最终返回的对象就是`__new__`方法创建的那个。

来一段官方文档解释

    `__new__`方法继承自`object.__new__(cls[,...])`, 调用创建类`cls`的实例。
    该静态方法接受一个`需要返回的对象的类`cls作为第一个参数，剩余参数都会被传递给对象构造表达式。
    返回值应该是一个新的对象实例（通常是cls的实例）

    典型用法是，以合适的参数调用超类的`__new__`方法`super(currentclass, cls).__new__(cls[, ...])`创建一个类的新实例，然后可以做一些修改，最后返回该实例

    如果`__new__`返回的是cls的实例，这个实例的`__init__()`会以`__init__(self[,...])`的形式调用，`self`即新创建的对象，后续的参数即是之前传给`__new__`的剩余参数

    如果`__new__`返回的不是cls的实例，那么实例的`__init__()`将不会被调用

    `__new__`主要意图是那些让不可变类型（例如 int, str, tuple）的子类可以自定义实例的创建。通常也会在自定义元类中被重写，从而达到自定义类的创建

这一段我自行翻译如上，原文见 [`object.__new__(cls[, ...])`](https://docs.python.org/2.7/reference/datamodel.html?highlight=__new__#object.__new__)

### `__init__`
[官方文档](https://docs.python.org/2.7/reference/datamodel.html?highlight=__new__#object.__init__) 翻译如下：

    在实例被(`__new__`)创建后调用，参数均将被传递给类构造表达式。
    如果该类所继承的基类也存在`__init__`方法，那么必须显式的调用该方法以确保实例的基类部分得到初始化。
    例如`Baseclass.__init__(self[,args...])`
    由于`__new__()`和`__init__()`共同作用创建的对象（前者创建，后者定制）
    所以`__init__`的返回值一般为`None`，如果不是，将会在运行时抛出`TypeError`

总之，继承类需要将新创建的对象，作为基类的初始化方法的第一个参数传入并调用，从而完成继承。

这个过程基类并未产生实例，只是将静态方法`__init__`绑定子类创建的对象调用，完成对象的"加工"

### 实例
举例来理解上面两段的官方文档
```python
class MyClass(object):
    def __new__(cls):
        print cls 

    def __init__(self):
        print self

obj = MyClass()
print(obj)
# <class '__main__.MyClass'>
# None
```
可以看到，`__new__` 方法返回的打印的cls就是定义的`MyClass`，未定义返回对象，所有obj的值为`None`。
又因为返回值非cls的实例，所以`__init__` 方法根本没被调用

继续实践
```python
class MyClass(object):
    def __new__(cls, arg):
        cls_instance = object.__new__(cls)
        cls_instance.arg = arg
        return cls_instance

    def __init__(self, arg):
        print(self.arg)
        self.foo = arg + ' world'

obj = MyClass('hello')
print(obj.arg)
print(isinstance(obj, MyClass))
# hello
# hello world
# True
```
这一段应该很好的解释了python类创建对象的过程
- 创建之前，首先寻找有没有实现`__new__`方法，有的话直接调用，随后根据条件判断是否调用`__init__`方法
- 如果没有重写同名类方法，则调用内部机制（这里以`object.__new__(cls[, ...args])`演示）创建类的实例并返回
- 如果返回的实对象是cls的实例，且定义了`__init__()`方法，随后该实例对象会被继续加工

#### 实例小结
- 类创建时`MyClass('hello')`, 传递给`__new__()`的参数是当前类和后续参数，传递给`__init__()`的参数是创建的实例和之前的后续参数
- `object.__new__(cls[, ...args])`会返回cls的实例
- `__init__()`方法只是接受实例对象做加工，并不创建也不返回该对象。
- 由上一条可知，如果创建的对象是不可变对象，那么`__init__`方法将完全不起作用

大多数情况下，是不需要使用到`__new__`方法的，它的应用场景如官方所言主要是两个
- 自定义一些immutable对象的创建
- 还有就是前面重点说到的自定义元类了

#### 动态修改类实例的创建
举例说明，创建一个字符串类，要求继承自`str`类，接受一个字符串参数，返回值为该字符串加上定制的前缀
```python
class MyString(str):
    def __init__(self, arg, prefix="hello "):
        print(isinstance(self, str))
        str.__init__(self, prefix + arg)

obj = MyString('world')
print(obj)
```
上述代码很遗憾不会有预期的效果。

分析上述代码的实际工作流程是
- 调用`MyString('world')`，不存在`__new__`方法，调用内部机制创建实例对象
- 内部机制不知道没关系（一说是type类），但`__init__`调用前实例对象必然已经创建，必然是`str`的实例，已然不可更改
- 再调用`__init__`方法，打印证实了之前的猜想

因此想要实现该需求，必须在字符串对象创建时就加上前缀，一旦创建完成便是不可变对象

```python
class MyString(str):
    def __new__(cls, arg, prefix='hello '):
        return str.__new__(cls, prefix + arg)

obj = MyString('world)
print(obj)
print(isinstance(obj, MyString))
print(isinstance(obj, str))
print(issubclass(MyString, str))
```
完成了功能，顺便验证了一点，`str.__new__(cls[,...])`方法不仅可以创建cls的实例，而且cls会从str继承

一般为了代码可维护性以及对多重继承的支持，会选择`super`的写法`super(MyString, cls).__new__(cls, prefix+arg)`

了解了`__new__`的用法,对比`__metaclass__`的用法,再次改写之前的例子
```python
class Foo(object):
    __metaclass__ = myMetaclass
    slogan = "you can't see me"
    def __init__(self):
        pass
```
实现如下
```python
class Foo(object):
    slogan = "you can't see me"
    def __new__(cls):
        if hasattr(cls, 'slogan'):
            cls.slogan = 'goodbye world'
        else:
            cls.slogan = 'hello world'
        return super(Foo, cls).__new__(cls)
    def __init__(self):
        pass

print(Foo.slogan)

foo = Foo()

print(foo.slogan)

# you can't see me
# goodbye world
```
可以看出二者和`__new__`不同的地方：
- `__metaclass__`会拦截类的创建，如果属性值存在则调用
- `__new__`只是在运行时修改类，未调用前是不会生效的，达到动态修改的目的

除此之外，`__new__`最常用的的用法是实现一个单例类
```python
class Singleton(object):
    def __new__(cls):
        if not hasattr(cls, 'instance'):
            cls.instance = object.__new__(cls)
        return cls.instance

obj1 = Singleton()
obj2 = Singleton()

obj1.key = 'value'
print obj1.key, obj2.key

print obj1 is obj2
```
#### 自定义元类
自定义元类, 以type元类为基类即可
```python
class MyMetachass(type):
    def __new__(cls, name, base, attrs):
        print(attrs['slogan'])
        attrs['slogan'] = 'guess what'
        return type.__new__(cls, name, base, attrs)

class Foo(object):
    __metaclass__ = MyMetachass
    slogan = "you can't see me"
    def __init__(self):
        pass

print(Foo.slogan)

foo = Foo()

print(foo.slogan)
```
效果和最开始的例子相同，也更符合面向对象的写法

## 总结
这一篇的内容有点多，本来也是想到哪儿写到哪儿，本来只想写metaclass的，一不小心没把持住……当然也还有很多没有展开，我猜太长自己都没耐心看，再次提炼一下本文中心思想。

- 元类(metaclass)就是实例为class的class
- type接收一个参数时是返回对象类型，接收三个参数时是一个元类。两种情况下的返回值均为type类
- 对象的`__class__`属性反映了创造它的类的类型，返回值是type类
- `__metaclass__`可以作为类属性，也可以作为模块属性，会拦截类的创建，会调用属性值来完成类的创建，属性值为一个可调用对象，接受的参数和type元类相同
- `__metaclass__`的效果会隐式的继承到子类，也会拦截子类的创建
- `__new__(cls[,...])`是`object`上的静态方法，作用是返回接受的第一个参数类的实例对象。可重写该方法动态的修改类
- `__init__(self[,...])`, 第一个参数为`__new__()`返回的对象，完成对象的初始化
- `__metaclass__`和`__new__()`可以联合使用，关键点在于将type看成一个类，可以从它继承进而生成定制元类
