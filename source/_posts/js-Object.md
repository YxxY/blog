---
title: JavaScript Object
date: 2017-07-11 20:50:38
categories:
- JavaScript
- Object
tags: 
- JavaScript
- Object
---

javascript里的对象是我一直没认真研究过的，每次看到Object的调用总觉得是高级语法。然而其实它一直都在那里，不难不易，只是我没有去了解。查了下MDN，仍旧是做下API层面的总结。
<!--more-->
# 用法

## 键值对初始化
    @param name String 
    @param value Any
其中对于name参数，必须是String。
如果不是String会被转化为String类型。1会转为'1'、null会被转为'null', undefined 会被转为'undefined', true将被转为'true'。如果是对象，则调用对象的toString方法返回String。
特别说明下`__proto__`。如果使用它作为name，对象不会创建一个以该字符串命名的属性。因为它存在默认值，指向Object.prototype。除非提供的value是一个对象或者null，否则默认值不会改变。
```js
var o = {};
var o = {a: 'foo', b: 42, c: {}};

var a = 'foo', b = 42, c = {};
var o = {a: a, b: b, c: c};

var o = {
  test: 5;
  get property() {return this.test},
  set property(value) {if(value>10) this.test = value}
};

var obj1 = {};
assert(Object.getPrototypeOf(obj1) === Object.prototype);

var obj2 = {__proto__: null};
assert(Object.getPrototypeOf(obj2) === null);

var protoObj = {};
var obj3 = {'__proto__': protoObj};
assert(Object.getPrototypeOf(obj3) === protoObj);

var obj4 = {__proto__: 'not an object or null'};
assert(Object.getPrototypeOf(obj4) === Object.prototype);
assert(!obj4.hasOwnProperty('__proto__'));
```
除此之外，ES6新增了语法特性
```js
/* Shorthand property names */
var a = 'foo', b = 42, c = {};
var o = {a, b, c};
/* o.a => 'foo' */

/* Shorthand method names */
var o = {
  property([parameters]) {}
};

/* Computed property names */
var prop = 'foo';
var o = {
  [prop]: 'hey',
  ['b' + 'ar']: 'there'
};
/* However, JSON.parse() will reject computed property names 
and an error will be thrown */
```
总之一个对象的初始化就是一个表达式，用来描述一个对象的初始化过程。对象由属性组成，属性是用来描述对象的。对象的属性值可以是基本数据类型值（即js六大基本数据类型，string、number、boolean、undefined、null、symbol），也可以是对象

## 用作构造器
    new Object([value])
    根据所给的value，返回一个对象包装器。
如果value的值为null或者undefined，会构造返回一个空对象；如果value是一个对象，则返回value；否则返回所给定value类型对应的对象。

# Object属性
## Object.length
    返回 1
## Object.prototype
    返回一个对象，包含的属性被其他对象继承，几乎所有对象都从这里继承。
    Object.prototype对象的属性不可枚举(not enumerable), 不可赋值(not writable), 
    不可更改或删除(not configurable),可通过 `in` 来确定是否存在某一属性。

### Object.prototype.constructor
    返回一个访问描述符(accessor property)，对象的构造函数
```js
var o = {};
o.constructor === Object;

var o = new Object;
o.constructor === Object;

var a = [];
a.constructor === Array;

var a = new Array;
a.constructor === Array;

var n = new Number(3);
n.constructor === Number;
```
其中对象的构造函数可以被改变，
但对基本类型数据number、boolean、string的构造函数进行更改不会生效。

    undefined、null不含有constructor属性
### `Object.prototype.__proto__`
    非标准规范，但被广泛实现。返回当前对象的原型对象，也是一个访问描述符，
    可以获取也可以修改，但是修改非常影响性能，不鼓励使用。
```js
let shape = function () {};
let p = {
    a: function () {
        console.log('aaa');
    }
};
shape.prototype.__proto__ = p;

let circle = new shape();

shape.prototype.a() /* aaa */
console.log(shape.prototype.hasOwnProperty('a')); /* false */
circle.a(); // aaa
```
### `Object.prototype.__defineGetter__()`
### `Object.prototype.__defineSetter__()`
    设置对象的getter和setter，非标准，可以被类方法Object.defineProperty代替
```js
var o = {};
o.__defineGetter__('gimmeFive', function() { return 5; });
console.log(o.gimmeFive); /* 5  */
o.gimmeFive = 10
console.log(o.gimmeFive); /* 5 */

o.__defineSetter__('value', function(val){this.test = val;});
o.value = 10
console.log(o.value); /* undefined */
console.log(o.test); /* 10 */

/* Using Object.defineProperty instead */
var o = {};
Object.defineProperty(o, 'foo', {
  get: function(val){
    return this.anotherValue;
  },
  set: function(val) {
    this.anotherValue = val;
  }
});

console.log(o.foo)          /* undefined */
console.log(o.anotherValue) /* undefined */
o.foo = 10
console.log(o.foo)          /* 10 */
console.log(o.anotherValue) /* 10 */
```
### `Object.prototype.__lookupGetter__()`
### `Object.prototype.__lookupSetter__()`
    返回对象的getter和setter函数，非标准，可以被类方法Object.getOwnPropertyDescriptor代替
```js
var obj = {
  get foo() {
    return Math.random() > 0.5 ? 'foo' : 'bar';
  }，
  set foo(value) {
    this.bar = value;
  }
};

/*  Non-standard and deprecated way */
obj.__lookupGetter__('foo');
/*  (function() { return Math.random() > 0.5 ? 'foo' : 'bar'; }) */
obj.__lookupSetter__('foo')
/*  (function(value) { this.bar = value; }) */

/*  Standard-compliant way */
Object.getOwnPropertyDescriptor(obj, "foo").get;
/*  (function() { return Math.random() > 0.5 ? 'foo' : 'bar'; }) */
Object.getOwnPropertyDescriptor(obj, 'foo').set;
/*  (function(value) { this.bar = value; }) */
```

### Object.prototype.hasOwnProperty()
    @param prop String|Symbol
    返回bool，判断该对象是否有某自定义属性，排除继承选项。

允许对象改写改方法，因此可用下面的方式调用更加安全
```js
({}).hasOwnProperty.call(obj, prop)
/* or this way */
Object.prototype.hasOwnProperty.call(obj, prop);
```

### Object.prototype.isPrototypeOf()
    @param object Object
    返回bool，判断函数调用对象是否在参数对象的原型链上
```js
var Test = function(){}
var test = new Test()

console.log(Test.prototype.isPrototypeOf(test));
/* true /*
console.log(Object.prototype.isPrototypeOf({}));
//true
```
### Object.prototype.propertyIsEnumerable()
    obj.propertyIsEnumerable(prop)
    判断对象属性是否可枚举

### Object.prototype.toString()
    返回字符串代表对象，Array, Number, Date 对象重写了该方法

### Object.prototype.valueOf()
    返回对象的基本数据类型值，一般用在需要将对象转换成基本类型值的场景，比如运算操作。
```js
function MyNumberType(n) {
    this.number = n;
}

MyNumberType.prototype.valueOf = function() {
    return this.number;
};

var myObj = new MyNumberType(4);
myObj + 3; // 7
```

# Object方法
相当于类方法

## Object.defineProperty()
    用法：Object.defineProperty(obj, prop, descriptor)
    返回传入的对象
可以给一个对象定义新属性，也可以修改已有属性，返回传入的对象
普通的对象属性赋值使得它的属性可以被枚举，属性值可以被修改还可以被delete删除。但该方法允许一些更细节的操作，比如通过该方法增加的属性默认不可枚举也不能删除。
@descriptor 参数有两种呈现风格，数据描述符(data descriptors) 和 访问描述符(accessor descriptors)。
共同点是，二者都是对象，因此不可缺省。且都有如下属性
    
    configurable：默认false，只有当为true时，该属性才能被修改和删除,
                修改指不能二次修改定义，和writeable为true时的修改值不冲突
    enumerable：默认为false，只有当为true时，才会在枚举时出现
数据描述符还有以下可选属性：

    value：属性值，可以是任何合法数据类型，对象或函数等等，默认undefined
    writable：默认false，只有当true时，属性值才能被赋值运算符改变

访问描述符有以下可选属性：

    get：属性的getter函数，函数返回值将作为属性值，默认为undefined
    set：属性的setter函数，默认为undefined

以上可选属性都不是必要的，如果要使用，有且只能使用二者中的一个。定义属性时，也要考虑对象的原型链继承，如需得到一个纯净的对象可以使用`Object.create(null)`

## Object.defineProperties()
    Object.defineProperties(obj, props)
```js
var obj = {};
Object.defineProperties(obj, {
  'property1': {
    value: true,
    writable: true
  },
  'property2': {
    value: 'Hello',
    writable: false
  }
  // etc. etc.
});
```

## Object.create(proto[, propertiesObject])
    返回一个新对象，它有着具体的原型和属性
    @param proto Object|null //创建的新对象的prototype
    @param propertiesObject Object|null 
    //可选，如果给定且不为undefined，那么它的值只能个对象，否则会报TypeError。
    该对象的自定义属性会以对应的属性名添加到新建的对象应Object.defineProperties()的第二个参数。
```js
var o = Object.create(Object.prototype)  
/* equals to: */
var o = {}

function Constructor() {}
o = new Constructor();
/* is equivalent to: */
o = Object.create(Constructor.prototype);

o = Object.create({}, { p: { value: 42 } });
o2 = Object.create({}, {
  p: {
    value: 42,
    writable: true,
    enumerable: true,
    configurable: true
  }
});
```
使用场景:

1. 创建一个空对象，`Object.create(null)`, 返回的对象无继承属性，连`__proto__`都没有
2. 实现对象的继承
```js
function MyClass() {
  SuperClass.call(this);
  OtherSuperClass.call(this);
}

/* 从一个类继承 */
MyClass.prototype = Object.create(SuperClass.prototype);

MyClass.prototype.myMethod = function() {
  // do a thing
};
```

## Object.assign()
    用法：Object.assign(target, ...sources)
    向target对象上增加一个或多个source对象自身的可枚举属性，
    如果有相同的属性，后续的对象会覆盖前面的属性，返回target对象
    assign的机制是赋值，调用source的getter，调用target的setter。而不是复制或者定义新属性。

assign的赋值不同于深度克隆
```js
let obj1 = { a: 0 , b: { c: 0}};

let obj2 = Object.assign({}, obj1);
obj2.a = 2;
obj2.b.c = 3;
console.log(JSON.stringify(obj1)); /* { a: 0, b: { c: 3}} */
console.log(JSON.stringify(obj2)); /* { a: 2, b: { c: 3}} */

/* Deep Clone */
obj1 = { a: 0 , b: { c: 0}};
let obj3 = JSON.parse(JSON.stringify(obj1));
obj1.a = 4;
obj1.b.c = 4;
console.log(JSON.stringify(obj1)); /* { a: 4, b: { c: 4}} */
console.log(JSON.stringify(obj3)); /* { a: 0, b: { c: 0}} */
```
基本类型会被包装成对象

```js
var v1 = 'abc';
var v2 = true;
var v3 = 10;
var v4 = Symbol('foo');

var obj = Object.assign({}, v1, null, v2, undefined, v3, v4); 
/* Primitives will be wrapped, null and undefined will be ignored.
   Note, only string wrappers can have own enumerable properties. */
console.log(obj); // { "0": "a", "1": "b", "2": "c" }
```

## Object.getOwnPropertyDescriptor()
    Object.getOwnPropertyDescriptor(obj, prop)
    @return a property descriptor if exists， undefined otherwise

## Object.getOwnPropertyDescriptors()
    Object.getOwnPropertyDescriptors(obj)
    返回对象所有属性描述符

## Object.keys()
    Object.keys(obj)
    返回数组，存储着对象自身的可枚举属性

## Object.getOwnPropertyNames()
    Object.getOwnPropertyNames(obj)
    返回一个数组，包含所有自身的属性名，包括不可枚举的属性
如果是数组对象，则返回所有的key以及'length'

## Object.getOwnPropertySymbols()
同上，只是返回的是symbol

## Object.getPrototypeOf()
    Object.getPrototypeOf(obj)
    返回对象的原型对象，是相对于`__proto__`的正规用法
    
ES5里参数如果不是对象会报TypeError,ES6会将参数包装成对象，并返回其原型对象

```js
    var Proto = function(){};
    var obj = new Proto();
    console.log(Object.getPrototypeOf(obj) === Proto.prototype); // true
    console.log(Object.getPrototypeOf(obj) === obj.__proto__); // true

    Object.getPrototypeOf('foo');
    // TypeError: "foo" is not an object (ES5 code)
    Object.getPrototypeOf('foo');
    // String.prototype                  (ES6 code)
```

## Object.setPrototypeOf()
    Object.setPrototypeOf(obj, prototype);
    修改对象的原型对象，返回该对象。
修改对象原型存在性能问题，应尽量避免，可以使用Object.create()创建一个新对象

## Object.is()
    Object.is(value1, value2);
    @return bool 判断两个值是否为同一个值，和`==`、`===`均不同

```js
Object.is('foo', 'foo');     // true
Object.is(window, window);   // true

Object.is([], []);           // false
Object.is({}, {});           // false
var test = { a: 1 };

Object.is(test, test);       // true
Object.is(null, null);       // true

Object.is(NaN, 0/0);         // true
Object.is(NaN, NaN);         // true
```

## Object.freeze()
    用法：Object.freeze(obj)
    返回一个对象处于不可修改的状态。修改不会生效，在Strict模式下会报错
    ES5对参数不是一个对象会报错，ES6则不会，返回输入值。

然而freeze对子对象的修改并不生效

```js
obj1 = {
  internal: {}
};

Object.freeze(obj1);
obj1.internal.a = 'aValue';

obj1.internal.a // 'aValue'
```

当然也是有办法做到递归freeze
```js
/* To do so, we use this function. */
function deepFreeze(obj) {
  /* Retrieve the property names defined on obj */
  var propNames = Object.getOwnPropertyNames(obj);
  /*  Freeze properties before freezing self */
  propNames.forEach(function(name) {
    var prop = obj[name];
    /*  Freeze prop if it is an object */
    if (typeof prop == 'object' && prop !== null)
      deepFreeze(prop);
  });
  /*  Freeze self (no-op if already frozen) */
  return Object.freeze(obj);
}

obj2 = {
  internal: {}
};

deepFreeze(obj2);
obj2.internal.a = 'anotherValue';
obj2.internal.a; // undefined
```

## Object.seal()
    Object.seal(obj)
    返回被封装的对象，阻止增加新属性，并将已有属性改成non-configurable，如果writable为true则属性值仍旧可以修改

## Object.preventExtensions()
    Object.preventExtensions(obj)
    返回被禁止增加新属性的对象，不影响对现在属性的修改

## Object.isExtensible()
    Object.isExtensible(obj)
    @return bool 判断一个对象是否可扩展
一般对象都是默认可扩展的，能增加新属性，也能被修改，但通过一些方法可禁止这些特性，如Object.preventExtensions(), Object.seal(), or Object.freeze().

# 总结
没啥可说的，理解了就不会觉得神奇。


参考：[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)