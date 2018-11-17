---
title: JavaScript箭头函数
date: 2017-05-18 20:50:38
categories:
- JavaScript
- Arrow-function
tags: 
- JavaScript
- 箭头函数
- this
---

js从ES6开始引入一种新函数，箭头函数。
它拥有无比简洁优雅的写法，不用显式的写`function`，确实方便不少。但并非适应所有的函数应用场景，这里结合资料和个人理解做个总结。
<!--more-->

    An arrow function expression has a shorter syntax than a function expression 
    and does not bind its own this, arguments, super, or new.target. 
    These function expressions are best suited for non-method functions, 
    and they cannot be used as constructors.

箭头函数相比声明式的函数表达式语法更加简洁，但它不绑定自己的<a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this">this</a>, <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments">arguments</a>, <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super">super</a>, 以及<a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new.target">new.target</a>, 箭头非常适合`非对象方法`的函数场景，而且也不能作为构造函数。

以上是我直接翻译的一段MDN资料,对于对象方法的使用场景特殊说明下：
不是说箭头函数不能用作对象方法，如果方法里不涉及this的调用，是完全可以的。
而如果使用了this，情况可能会和你想的不一样，这个后续会说到。

## 基本用法
```js
(param1, param2, …, paramN) => { statements }
(param1, param2, …, paramN) => expression
/*当函数体只有一个表达式时，可以省略大括号， 
等价于: (param1, param2, …, paramN) => { return expression; } */

(singleParam) => { statements }
singleParam => { statements }
/*当只有一个参数时,可以省略括号 */

() => { statements }
/* 无参数时，括号不可省 */

params => ({foo: bar})
/* 给函数体加上括号，表示函数返回一个对象 */

(param1, param2, ...rest) => { statements }
(param1 = defaultValue1, param2, …, paramN = defaultValueN) => { statements }
/* 支持Rest参数和默认参数 */

let f = ([a, b] = [1, 2], {x: c} = {x: a + b}) => a + b + c
f()  //6
//支持带参数列表的解构赋值

```
## 从this说起
this的用法，在面向对象编程的领域里显得非常的另类。
在箭头函数出现之前，每一个新的函数，都定义了自己的this这个特殊变量。
如果是作为构造函数，this指代一个新的对象; 
如果是在strict模式下的函数调用，this被定义为undefind; 
非strict模式则指向全局对象(window或global); 
如果是被当做对象的方法调用，this则指代调用该方法的对象。

然而对于箭头函数，它不绑定自己的this。

```js
var age = 5; /* nodejs 需写成 global.age = 5  */ 

function Foo(){
    /* this.age = 0;//this 指向构造函数Foo的实例 */
    var test= function() {
        /* 非strict模式下指向全局对象，非Foo的实例 */
        console.log(this.age);
    };
    
    test();
}

var p = new Foo(); 
//5

```

如想让以上这段代码输出为0怎么办呢？
在ES 3/5里，this可以被赋值给其他变量，于是可以有如下的这种办法

```js
function Foo() {
    var that = this; /* 一开始捕获this，将它存为that */
    that.age = 0;

    var test = function() {
        console.log(that.age);
    };

    test();
}

var p = new Foo();
//0
```
当然还可以使用绑定函数，给函数绑定this的指向，如下
```js
var age = 5; /* 如果是nodejs，需写成global.age = 5 */
function Foo() {
    this.age = 0;

    var test= function() {
        console.log(this.age);
    };

    var test = test.bind(this);/* 也可以用call或者apply 
    test.call(this, null); 
    test.apply(this, null); */

    test();
}

var p = new Foo();
//0
```
但是，这个场景刚好适合箭头函数登场
```js
var age = 5;
function Foo() {
    this.age = 0;

    var test = ()=>{
        console.log(this.age);
    }
  
    test();
}

var p = new Foo();
//0
```
因为箭头函数没有自己的this上下文，所以它会根据外围的上下文获取this，
在这里就是我们希望所希望的Foo实例对象。

    总之正常函数里使用this时，指向虽然不确定，但是至少有迹可循，
    但在箭头函数里使用，就是真的不确定，因为它是绑定在外层作用域的。
    倘若不使用this，只亲睐它的简洁语法，那当然就没有这些问题了。
    一旦使用，就要考虑清楚它是从哪里继承。

举最后一个例子,验收测试
```js
function foo() {
    return () => {
        return () => {
            return () => {
                console.log(this);
            };
        };
    };
}

new foo()()()()     /* this ? */
foo()()()()         /* this ? */
```

这段代码的两个结果分别是什么呢？可以自己验证下。
答案如下:
第一个this指向foo()的实例对象
第二个指向全局对象


## 依次类推
说完this，同理对一开始说到的arguments, super等也是适用的
参考下面这段代码
```js
function Foo() {
   var test = ()=>{
        console.log("args:", arguments);
   }
   test()
}

new Foo(2, 4, 6, 8);
// args: {'0':2, '1':4, '2':6, '3':8}
```
箭头函数没有自己的arguments, 它就取Foo()的arguments。和不绑定this一个道理。

## Tips
箭头函数的箭头和参数之间不能有换行，否则报语法错误
```js
var func = ()
           => 1; 
// SyntaxError: expected expression, got '=>'
```

箭头符号不是一个操作符，但会被误认为操作符，和其他操作符混用时需注意
```js
let callback;

callback = callback || function() {}; /* ok */

callback = callback || () => {};      
/* SyntaxError: invalid arrow-function arguments */

callback = callback || (() => {});    // ok
```


参考资料：<a href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions">箭头函数</a>