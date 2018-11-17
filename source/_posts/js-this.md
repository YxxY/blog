---
title: js中的this、bind、call、apply
date: 2017-10-11 20:30:38
categories:
- JavaScript
- This
tags: 
- this
---
发现一个很有意思的现象，js中的`this`、`bind`、`call`、`apply`单独拿出来都知道用法，但只要两个的组合出现时就有点蒙，三个的组合出现时开始怀疑自己是否还没入门……

我想大概原因还是没有认真做过总结吧，之前已经总结过一次js中this的用法，不过那次是着重总结箭头函数中的this。这次把这几个经常见到的整合在一起再总结一次。
<!--more-->
# 总述
- this，指向一个对象，不同情况下不同, 如果是函数调用，this指向函数运行时的context，即上下文对象
- bind，函数方法，调用后返回一个创建的新函数，第一个参数为传递给创建函数的this变量，后续参数为创建函数参数，会先于调用函数参数传入
- call，函数方法，调用后执行该函数，第一个参数为传递给该函数的this变量，后续参数为函数参数
- apply，函数方法，调用后执行该函数，第一个参数为传递给该函数的this变量，第二个参数可省略，但存在的话必须为一个数组或类数组

# this
现在随手一搜关键字`js`, `this`, 能找到无数介绍资料，很多人总结的都很好，很详尽，我也来一份我自己的总结。

## this出现的位置
一般this都是出现在函数内部，但在函数外也是可以的，例如:
- 在浏览器的console里敲
```js
this === window //true
```
- 在nodejs脚本里写
```js
console.log(this === exports) //true
console.log(this) //{}

a = 8
function test(){
    console.log(this === global)  //true
    console.log(this.a)  //8
}
test()
```
- 但strict 模式下脚本结果会有不同
```js
'use strict'

function test(){
    console.log(this) //undefined
}
test()
```
- 在node REPL模式下输入
```js
console.log(this === global) //true
```

在普通函数内部，this默认指向全局变量，浏览器里对应就是`window`，node中为`global`。strict模式下则强制指向`undefined`。
但函数作为`构造函数`或`对象方法`时情况则不同。但都符合一个原则，统统指向函数执行时的上下文对象
```js
//构造函数
function Test(name){
    this.name = name
}

var test = new Test('test')
console.log(test.name) //test

//对象方法
var obj = {
    name: 'test',
    test: function(){
        console.log(this.name)
    }
}
obj.test() //test
```
构造函数和对象方法的上下文对象都很清楚，就是调用函数的对象。
这里再复习下箭头函数，如果是箭头函数的调用呢
```js
(()=>{
    console.log(this);
})()
```
对于箭头函数，它没有自己的上下文，依赖于父上下文，即向外最近的this指向就是它的指向，那这里就相当与在脚本里直接打印this了，结果为`{}`,指向`exports`包对象本身，如果是浏览器则是`window`对象。

## 函数内部的函数this
```js
name = '你猜'

function Test(name){
    this.name = name
    this.foo =  function(){
        return this.name
    }
    function bar(){
        console.log('#'+this.name);
    }
    bar()
}

var test = new Test('test')  //#你猜
console.log(test.name)  // test
console.log(test.foo()) // test
```
以上代码结果会是什么呢？
涉及的三个函数Test，bar和foo，它们内的this是同一对象么？
答案是否定的，非匿名函数的this指向都是独立的，情况和之前说的一样，作为普通函数和构造函数及对象方法是不一样的。

# call和apply
## 用法
call和apply区别只是第二个参数不同，放在一起说。
之前说this在函数中的指向虽然不总是一致，但也是有规律的，例如定义一个对象，那么它的方法函数内部的this，一般情况下都是指向它本身的，没有什么问题。还是之前的例子
```js
var obj = {
    name: 'test',
    test: function(){
        console.log(this.name)
    }
}
obj.test() // test
```
但这是新定义了一个对象，也想使用test方法呢？你会想到直接赋值不就可以了么，如下：
```js
name = '你猜'
var foo = {name:'foo'}
foo.test = obj.test
foo.test()
```
结果行不行呢？哈哈，你猜？
答案是可以滴！属性赋值，函数没有立即执行，运行中的上下文是可以改变的，当调用`foo.test()`时，函数的this会指向foo对象的。
但注意如果是
```js
var test= obj.test
test()
```
这样的话test又回归一个普通函数了，this的则指向全局变量。

相比较而言，call和apply提供了较为简洁的方法
```js
name = '你猜'
var foo = {name:'foo'}
obj.test.apply(foo)
```
即手动指定函数内部的上下文，让不确定变得确定。
这样做的好处是什么呢，会发现call和apply实现了js中的所有对象的方法共享，即对象方法不再只限定为该对象自身调用，其他任何对象都可以通过传递this的方式改变函数执行时的上下文从而实现一处定义，多处调用。

## this语法糖
到这里几乎把函数调用的方式都举例了，总的来说就三种：
- func()
- obj.func()
- func.call() || func.apply()
通常使用的都前两种，然而事实是前两种都是以第三种调用模式实现的语法糖，本质还是第三种调用，这样做只为让语法更简洁。

    因此func() 等价于 func.call(this)，obj.func() 等价于 obj.func.call(obj)

this即传入的上下文对象，隐式情况下会有之前所提到的种种默认值，显示情况则可以手动指定

# bind
## 永久改变this指向
bind和call及apply类似，也可以动态修改函数执行的上下文，但又有不同，如果说call和apply是“借尸还魂”，那么bind就是“影分身”了，它创建新函数，产生独立的对象。
```js
var obj = {
    name: 'test',
    test: function(){
        console.log(this.name)
    }
}

var test =  obj.test
test()
```
这是刚才的一个例子，结果肯定不是’test'，原因之前解释了，复制语句让test变成一个普通函数，运行时上下文对象会指向全局变量。
当然可以使用
```js
test.apply(obj)
```
这样的方法动态修改上下文指向。但bind可以提供永久改变上下文的方法，即：
```js
var test =  obj.test.bind(obj)
test() // test
```
这个时候test函数的this指向就固定了，指向obj。
```js
var test =  obj.test.bind(obj)
test()  //test
test.apply({name:'xxx'}) // test，apply传入的this未生效
obj.name = 'xxx'
test() //xxx， 随上下文对象变化
```
此时apply方法的动态修改将不再生效，永久指向obj，obj变化时它才会变化。
## bind传参
bind函数也可传参，而且还有点特别，这里引用mdn的例子作简要说明
```js
function list() {
  return Array.prototype.slice.call(arguments);
}

var list1 = list(1, 2, 3); // [1, 2, 3]

// Create a function with a preset leading argument
var leadingThirtysevenList = list.bind(null, 37);

var list2 = leadingThirtysevenList(); 
// [37]

var list3 = leadingThirtysevenList(1, 2, 3);
// [37, 1, 2, 3]
```
## tips
bind 的第一个参数如果不是对象，会被包装成对象类型传入。

# 组合使用
其实刚才举例大多已经是组合使用，都是与this的组合，下面举两个个三者组合的例子。
## fn.bind.apply
```js
function add(a1, a2){
    console.log(a1+a2)
}

function test(fn){
    return fn.bind.apply(fn, arguments)
}
var bar = test(add, 1, 2)
console.log(bar) // '[Function: bound add]'
bar() // 3
```
apply的作用是调用一个函数，可以修改运行时的上下文对象。
在这里apply调用的同样是一个函数`fn.bind`, 指定它的this对象指向`fn`，并提前传入参数`arguments`。
bind函数的作用是返回一个新函数，于是`fn.bind.apply(fn, arguments)`返回了一个新函数。通过打印可以看到，打印上文的`bar`显示`[Function: bound add]`,此时`bar`已经是独立的函数，并且根据之前传参的介绍，参数1,2已经提前传入，所有后续调用bar函数时无需再次传参，直接调用即可。

## fn.apply.bind
都知道函数的arguments对象其实并不是一个标准的数组对象，往往需要将它转化为数组使用
```js
// 通常做法是用apply或call实现
function test(){
    console.log(Array.isArray(arguments)) //false
    var arg = Array.prototype.slice.apply(arguments)
    console.log(Array.isArray(arg)) //true
}

//bind方法实现
var unboundSlice = Array.prototype.slice;
var slice = Function.prototype.apply.bind(unboundSlice);

function test(){
    console.log(Array.isArray(arguments)) //false
    var arg = slice(arguments);
    console.log(Array.isArray(arg)) //true
}
test()
```
这里的`fn.apply.bind`的使用返回一个新的`bound apply`函数，它的上下文对象为`unboundSlice`。也就是说原来的apply函数必须依赖函数对象才能调用，现在可以独立调用了，此时的slice是一个包含特定this的apply方法。

## fn.bind.bind
由第二个例子，我生生联想到这个例子，发现真的可行，足以证明我似乎好像真的理解了，差点被自己感动……
```js
var slice = Array.prototype.slice
var boundBind = Function.prototype.bind.bind(Function.prototype.apply)
var boundApply = bound(slice)

function test(){
    console.log(Array.isArray(arguments)) //false
    var arg = boundApply(arguments);
    console.log(Array.isArray(arg)) //true
}
test()
```
正常人应该不会这么写的，主要为了理解，具体可自行分析。

# 总结
- this实际上是在函数被调用时建立的一个绑定,具体指向跟上下文有关
- apply、call、bind都是`Function.prototype`对象的属性，都是函数方法，都可以动态修改函数运行时的上下文对象
- apply、call的调用会立即运行该函数
- bind的调用会创建一个新的函数
- apply传参必须为数组或类数组
- call和bind传参为位置参数
- bind函数传参会先于新函数调用时的参数传入
