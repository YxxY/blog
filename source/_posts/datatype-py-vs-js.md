---
title: 基本数据结构：Python vs Javascript 
date: 2018-2-22 21:35:27
categories:
- Common-sense
- Data-structure
tags: 
- javaScript
- python
---
写多了nodejs再回来写python提笔就忘，说出来还被人说代码量少……嗯，无法反驳。
仔细想了下，徒手写代码到底困难在哪儿，想来大概是数据结构和基本语法了。
把这两种语言放一块儿对比着来总结下，主要就数据结构和基本语法两部分，备忘。
<!--more-->
## 数据类型
### js
六大基本类型以及引用类型

基本类型：number, string, boolean(true/false), null, undefined, Symbol
引用类型：Object

对象还可细分为 `Array`, `Functio`n, `Date`, `Math`等

ES6新增了`Set`和`Map`对象等

### py
基本类型和集合

基本类型：number, string, boolean(True/False), None(空值)
集合：list, tuple, set, dictionary

## 变量
### py
python的变量不用特殊声明，除非在局部作用域使用全局变量需要使用`global`声明

另外，python里一切皆对象，变量的赋值相当于将name和object绑定在一起
```py
a= 3
```
- 创建name a
- 创建object 3
- 将name a 关联到 3这个object 

以后就可以用a来调用3这个object

### js
相比python，js就要显得麻烦一些
#### 变量声明

ES6新增`let`,`const`声明关键字
- `let`，块级作用域
- `var`，函数作用域
- `const`， 声明变量值不可更改，对象则是指针的指向不可变

```js
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1

for (let i = 0; i < 3; i++) {
  let i = 'lalala' //不会报错，for循环体和循环条件是两个作用域
  console.log(i) 
}
console.log(i); //ReferenceError: i is not defined
```

- `var`可重复声明，后续变量值会覆盖前面
- `let`和`const`均不可重复声明，否则会报错

#### 变量提升
- `var`存在变量提升，
- `let`和`const`不存在变量提升
- 函数声明也会得到提升
```js
console.log(foo); // 输出undefined, 声明未赋值
var foo = 2;

console.log(bar); // 报错ReferenceError
let bar = 2;

console.log(bar); // 报错ReferenceError
const bar = 2;
```
再举一个例子
```js
var tmp = new Date();

function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world';
  }
}

f(); 
```
输出不是当前日期，而是`undefined`, 因为函数内部tmp的声明被提升

虽然日常代码不会这么写，但非常具备迷惑性

#### let和const的暂时性死区（Temporal Dead Zone）
只要`块级作用域`内存在let或const命令，它所声明的变量就`绑定`（binding）这个区域，不再受外部的影响。
```js
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```
如果注释掉let的声明，这段代码是没有问题的

但是块级作用域内let又声明了一个局部变量tmp，导致后者绑定这个块级作用域，所以在let声明变量前，对tmp赋值会报错

    总之，暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量

## 数据类型判断
### js
- `typeof`, 运算符，返回表达式数据类型的全小写`字符串`
```js
typeof 123       // 'number'
typeof NaN       // 'number'

typeof 'abc'     // 'string`

typeof undefined // 'undefined'

typeof null      // 'object'

typeof true      // 'boolean'

typeof {}        // 'object'
typeof []        // 'object'
typeof function(){}    // 'function'

typeof unknownVariable // 'undefined'
```
数组的判断使用ES6里的新增方法`Array.isArray`, 接受一个参数返回布尔值。

或者自行如下实现
`Object.prototype.toString.call(arg) === '[object Array]';`

`NaN`的判断使用`Number.isNaN`
### py
- `type(var)`,和已知变量类型对比
```python
type(myInt) is type(1)
type(myFloat) is type(.1)
type(myLong) is type(1111111111111)
type(myBool) is type(True)
type(myStr) is type("a")
type(myList) is type([1])
type(myTuple) is type((1,))
type(mySet) is type(set([1]))
type(myDict) is type({1:2})
```
- `isinstance(object, class-or-type-or-tuple)`, 返回布尔值
```python
isinstance(myInt, int)
isinstance(myFloat, float)
isinstance(myLong, float)
isinstance(myBool, bool)
isinstance(myStr, str)
isinstance(myList, list)
isinstance(myTuple, tuple)
isinstance(mySet, set)
isinstance(myDict, dict)
# 第二个参数使用tuple类型
isinstance(myDict, (list, tuple, set, dict))
```
- 内置库，types
```python
import types

type(myInt) is types.IntType
type(myFloat) is types.FloatType
type(myLong) is types.LongType
type(myBool) is types.BooleanType
type(myStr) is types.String
type(myList) is types.ListType
type(myTuple) is types.TupleType
# types库中没有对应的set类型，需要前两种方法
type(myDict) is types.DictType
```

## 判断相等
### js
使用`===`,不建议使用`==`来自找麻烦
```js
null === null            //true
undefined === undefined  //true
{} === {}                //false
NaN === NaN              //false
```
### py
判断相等，python和js出入较大

python的对象包含三要素:id，type，value
- `is`，id和value都相等
- `==`，value相等
- `!=`，value不相等
```python
a = b = [1,2,3]
c = [1,2,3]
d = {'a':1} 
e = {'a':1} 

print a == b  # True
print a == c  # True !!!
print a is b  # True
print a is c  # True
print d == e  # True !!!
print None is None # True
print None == None # True
```
这里主要说明了`is`和`==`的区别。同时也引出了第二个问题，对象相等

另外同一类的实例对象，一般是不相等，因为内存地址不同

但可以重载`__eq__`方法实现定制

与之相反的是`__ne__`方法
```python
class Foo(object):
    def __init__(self, name):
        self.name=name
        
    def __eq__(self, other):
        return self.name == other.name

print Foo('zhangsan') == Foo('lisi')     # False
print Foo('zhangsan') == Foo('zhangsan') # True
```
从而不难理解为什么python里引用类型也会出现相等的情况了，其实只是单纯的值相等而已
## 假值
### js
- `false` (boolean)
- 空字符串
- 数值为`0`
- `undefined`
- `null`
- `NaN`
### python
- `False` (boolean)
- 空序列，包括空字符串, [], {}, ()
- 数值为`0`
- `None`


## 判断及循环
### js
```js
// if/else
if(){

}else if{

}else{

}

//while
while(){

}

//for
for(var i in obj){  //i: index or key
    
}

for(var x of array_obj){ //x: value

}

for(var i=0;i<10;i++){

}
```
### python
```python
# if/else
if cond:
    statement
elif cond:
    statement
else
    statement

# while
while cond:
    statement

# for, 任何可迭代的对象都可以for循环
for value in array_list:
    print value

for index in range[len(array_list)]
    print index

d = [(1,2),(2,3),(3,4)]  
# 迭代二元list 
for x, y in d:  
    print x, y

d = { 'Adam': 95, 'Lisa': 85, 'Bart': 59, 'Paul': 74 }  

# 迭代dict的键，相当于将字典转换成key的list，相比iterkeys更费内存 
for x in d.keys():  
    print x  
  
# 也可以采用这种方式迭代  
for x in d.iterkeys():  
    print x  
  
# 迭代dict的值  
for x in d.values():  
    print x  
  
for x in d.itervalues():  
    print x  
  
# 迭代键值对  
for k, v in d.items():  
    print k, ":", v  
  
for k, v in d.iteritems():  
    print k, ":", v  
```

## 常用数据结构
### js
#### 数组 - `[]`
- 索引越界访问返回`undefined`

常用属性方法：
- `length` 返回数组长度
- `push(element1[, ...[, elementN]])`,
- `unshift(element1[, ...[, elementN]])`
- `pop()`
- `shift()`
- `sort([compareFunction])`, 默认从小到大
- `includes()`
- `join()`， 返回字符串，默认逗号分隔

#### 对象 - `{}`
- key-value赋值，点运算符或`[属性名]`均可
- 访问不存在的属性返回`undefined`
- `key`一般为字符串
- for循环取到的是key值
- ES6 支持对象属性解构
- `Object.keys(obj)`,返回对象可枚举属性的字符串数组
- `Object.values(obj)`,返回对象可枚举属性值的数组

### py
#### list - `[]`
- 索引越界访问返回`IndexError`
- `-1`做索引可以取到最后一个元素，依次类推
- 获取长度使用`len(list_var)`

常用属性方法
- `append(element)`, 追加元素到末尾
- `insert(index, element)`, 查人元素到指定位置
- `pop(index)`, 删除指定位置的元素，返回该元素

列表生成器
```python
l = [x * x for x in xrange(10)]
```

将中括号`[]`换成括号`()`,即返回一个生成器
```python
g = (x * x for x in xrange(3))
print next(g) # 0
for x in g:
    print x
# 1
# 4
```

#### tuple - `()`
和list类似，但是tuple一旦初始化就`不能修改`（指向）

也因此没有动态修改的方法，只有通过索引来访问值

定义一个tuple使用`t=()`
但是定义一个元素的tuple要使用`t=(1,)`,否则会返回`数字1`

#### dict - `{}`
- 访问通过索引或者`get`方法
    - 通过索引访问的key不存在会报`keyError`
    - 通过get方法，返回None或指定值
```python
>>>d['tom'] = 7
>>>d['jerry']
keyError
>>>d.get('jerry', -1)
-1
```
- `key`值可以是任意类型，但必须是`不可变对象`
- `pop(key)`, 删除指定key，返回value

### set - `([])`
和`dict`类似，但set只是一组key的集合，key值不重复，且不包含value

重复值会被过滤掉，且不保证顺序，是数学意义上的无序不重复元素的集合
- 初始化
```python
### 直接定义
s = {1,2,2,3}     #set([1,2,3])

### 接受一个list参数
s= set([1,2,3,2]) #set([1,2,3])
```
- set的元素只能是不可变对象，可变对象无法哈希化确定唯一值
```python
s = {(1,2,4)}    # ok
s = {(1,2,[3])}  # typeError
```
常用方法
- `add(key)`, 添加新元素，重复添加无效
- `remove(key)`, 删除元素
- `s1 & s2`, 取交集
- `s1 | s2`, 取并集