---
title: Promise
date: 2017-12-21 21:15:27
categories:
- JavaScript
- Promise
tags: 
- javaScript
- promise
---

作为es6的新特性，`promise`的出现，使得`callback hell`有了优雅的解决方案，掌握promise的使用无疑是一种技能提升。本文基于《ECMAScript 6 入门》和诸多碎片化的阅读以及实践中遇到的问题做一次全面总结。
<!--more-->

## 背景
Promise 是异步编程的一种解决方案，比传统的解决方案`回调函数和事件`更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。
```js
//Promise对象是一个构造函数
var promise = new Promise((res, rej)=>{
    if (/* 异步操作成功 */){
        res(value);
    } else {
        rej(error);
    }
})
})
```
## 参数
- 初始化实例时的参数是一个函数。
- 函数有两个参数，也均为`函数类型`，可分别在成功、失败时调用，作为对象状态改变的标志
- 调用`res`或`rej`时，最多只能传`一个参数`给后续回调函数，多个参数时后续参数不会被接收。可传递一个数组或对象解决多个参数的需求。

## 特点
- Promise对象包裹了一个异步操作，对象有三种状态：`pending`（进行中）、`resolved`（已成功）和`rejected`（已失败）。对象状态是私有属性，不受外界影响，只有内部操作才能改变。
- Promise对象初始状态为`pending`，当异步操作完成时，调用`res()`，即更改状态`pending -> resolved`; 异步操作失败时，调用`rej()`，即更改状态`pending -> rejected`
- 有且仅有上述两种状态改变的场景，且状态一旦改变就不会再变化。
- 状态变化时按注册顺序调用回调函数，实现异步调用的同步效果
- 调用`res`或`rej`并`不是终结promise`内部代码的执行，而是发送异步事件的结果到事件循环，内部后续代码反而会先于异步执行。要实现终结效果加上`return`即可。
- Promise实例化内部代码都是立即执行，后续处理都是回调，都会在本轮事件循环结束时才执行
- 当状态是pending时无法得知状态会如何改变，取决于异步操作结果
- 由于状态变化后无法改变且一直保持，对改变状态后的promise对象添加回调函数也会立即执行，不会像事件监听那样错过就不会再触发。
- 由于上一条的特性，多个回调函数时无法中途取消执行
- 当有内部错误，且没有注册`rejected`状态处理函数时会报错，一般建议都注册错误处理函数
- 回调函数默认返回新的Promise对象，也可以手动指定promise对象返回

## 用法
### Promise.prototype.then() 
Promise实例生成以后，可以用继承的`then`方法分别指定resolved状态和rejected状态的回调函数。`then`方法`返回的是一个新的Promise实例（注意，不是原来那个Promise实例）`
```js
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```
then方法在`调用它的Promise对象`的状态发生变化，才会被调用。
### Promise.prototype.catch() 
`Promise.prototype.catch`方法是`.then(null, rejection)`的别名，用于指定发生错误时的回调函数，属于语法糖范围，使代码更简洁。
catch有如下特点：
- promise对象状态变为`rejected`时调用
- `then`方法内部运行错误也能被捕获，但状态改变后再抛出错误不会被捕获
- promise对象的错误能冒泡传递，所有catch能获取整个流程的所有错误
- catch返回的也是新的promise对象，因此后续仍可以使用then方法，只是后续then状态变化或内部出错，不会再被之前的catch捕获
- catch内也会报错，也会被当做一个未处理的错误，此时再之后再追加一个catch也能捕获该错误。

因为catch的强大功能，所有一般不适用`then`函数的第二个参数去捕获错误，而总是使用catch

### Promise.resolve()
对象静态方法，作用是将现有对象转为Promise对象
### 参数类型
- `promise实例`，不做修改，直接返回
- `thenable`对象(具有then方法的对象), 会立即执行对象的then方法

```js
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
  console.log(value);  // 42
});
```
- 参数不是具有then方法的对象，或根本就不是对象，则返回一个新的resolved状态的 Promise 对象，参数向后传递。

```js
const p = Promise.resolve('Hello');

p.then(function (s){
  console.log(s)
});
// Hello
```
- 无参数。直接返回一个resolved状态的 Promise 对象
- 仍是最多只接受一个参数，多余的参数不会被传递

## Promise.reject()
类似`Promise.resolve()`, 不过返回的promise默认`rejected` 状态。
- 仍然最多只接受一个参数
- 不同的是，传递的参数`都会被原封不动的向后传递`

## Promise.all()
用于将多个 Promise 实例，包装成一个新的 Promise 实例。
- 参数必须是有 Iterator 接口，且返回的每个成员都是 Promise 实例的类型，通常为数组，成员为Promise对象
- 参数中如果存在不是promise对象的，会调用`Promise.resolve`方法转化
- 返回值为一个新的Promise对象

返回对象的状态分两种情况：
1. 只有参数中的promise对象有一个变成rejected，此时第一个被reject的实例的返回值，会传递给该对象的回调函数
2. 参数所有实例状态均变为resolved，返回值对象才会改变并同步，返回值一起传递给对象的回调函数。

参数中的promise对象没有自己的catch方法时，就会调用返回值对象的catch方法

## Promise.race()
和all方法类似，但状态变化有区别，只要有一个实例率先改变状态，返回结果的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给返回结果的回调函数。

## 扩展
基于以上对Promise对象的理解，很容易联想到可以做一些扩展功能
- `done()` - 针对catch的补充，如果catch内部出错，希望也能捕获错误，而不是后续继续增加catch
- `finally()` - 有些操作希望无论异步操作是成功还是失败都执行
- `try()` - Promise内部基本都是一个异步函数，希望能能接受一个同步函数为参数，同时能让该函数立即执行，返回值为promise

```js
//第一版done, 其实只是简单包装了catch, 将错误抛向全局。
Promise.prototype.done = function(){
    this.catch(err=>{
        process.nextTick(()=>{
            throw err
        })
    })
}

//第二版，自定义错误处理函数
//其实更加简单了，给done传递错误处理函数即可，`.done(err=>console.log(err))`
Promise.prototype.done = function(rej){
    this.catch(rej)
}

//第三版，增加一个成功时执行操作，失败时错误上抛
Promise.prototype.done = function(res){
    this.then(res)
        .catch(err=>{
            process.nextTick(()=>{
                throw err
            })
        })
}

//第四版，自定义成功或失败时的操作
Promise.prototype.done = function(res, rej){
    this.then(res, rej)
        .catch(err=>{
            process.nextTick(()=>{
                throw err
            })
        })
}
```

- `finally`的最关键点是不能在catch之后，因为放在catch之后，失败时被catch捕获，返回新的promise对象，这时状态是无法获取到的。因此这并不是想python里那么纯粹的finally。

```js
Promise.prototype.finally = function(f){
    return this.then(value=>{
        f()
        return Promise.resolve(value)
    }, err=>{
        f()
        return Promise.reject(err)
    })
}

```

- `try`最简单的实现，是在最外层再包装一个Promise，因为promise初始化过程都是立即执行的

```js
const f = () => console.log('now');
(
  () => new Promise(
    resolve => resolve(f())
  )
)();
console.log('next');
// now
// next
```
之所以需要这么麻烦的实现这个需求，是因为很多场景下流程的第一步都是同步生成一个Promise对象，然后用then和catch控制流程，
然而如果在最开始生成Promise之间程序有内部错误，这个错误是不会被catch捕获到的，所有这时try的实现很有必要，将第一步同步生成Promise对象的操作包裹起来，即使出错也会被catch捕获处理。

## Tips

### promise 的链式调用。

提起链式调用我们通常会想到通过 return this 实现，不过 Promise 并不是这样实现的。promise 每次调用 `.then 或者 .catch 都会返回一个新的 promise`，从而实现了链式调用。

```js
Promise.resolve(1)
  .then((res) => {
    console.log(res)
    return 2
  })
  .catch((err) => {
    return 3
  })
  .then((res) => {
    console.log(res)
  })

//1
//2
```

### promise 的 .then 或者 .catch 可以被调用多次
但这里 Promise 构造函数只执行一次。或者说 promise 内部状态一经改变，并且有了一个值，那么后续每次调用 .then 或者 .catch 都会直接拿到该值。

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('once')
    resolve('success')
  }, 1000)
})

promise.then((res) => {
  console.log(res)
})
promise.then((res) => {
  console.log(res)
})

//success
//success
```

### then 或者 .catch 中 return 一个 error 对象并不会抛出错误

```js
Promise.resolve()
  .then(() => {
    return new Error('error!!!')
  })
  .then((res) => {
    console.log('then: ', res)
  })
  .catch((err) => {
    console.log('catch: ', err)
  })

// then: Error: error!!!
//     at Promise.resolve.then (...)
//     at ...
```

因为返回任意一个非 promise 的值都会被包裹成 promise 对象
即 `return new Error('error!!!')`

等价于`return Promise.resolve(new Error('error!!!'))`

传递错误的方式有
- `return Promise.reject(new Error('error!!!'))`
- `throw new Error('error!!!')`


### .then 或 .catch 返回的值不能是 promise 本身，否则会造成死循环。

```js
var promise = Promise.resolve()
  .then(() => {
    return promise
  })
//TypeError
```

### .then 或者 .catch 的参数期望是函数，传入非函数则会发生值穿透

```js
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)
//1
```



