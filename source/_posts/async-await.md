---
title: async/await
date: 2018-2-28 21:15:27
categories:
- JavaScript
- Async-await
tags: 
- javaScript
- async
- await
---

async/await, 号称js异步的终极方案……虽然似乎只是语法糖，但确实减少了代码量，让代码同步化，一直拖着没用，但不学不行啊
<!--more-->

## 基本语法
### tips
- `async`修饰放在函数声明前，表示函数里有异步操作
- `async`函数的返回值会被封装为`promise`对象
- `await`只能出现在`async`函数中，表示后面的表达式需要等待结果，
- `await`后面的返回值一般是promise对象，如果不是也会被`Promise.resolve()`转换
- `await`后面的promise对象状态如果变为`reject`,会有`return` 的效果，不再执行后续语句
- 如果希望前一个异步失败不影响后面的状态，需要将`await`语句用`try...catch`包裹

基本示例：
```js
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
  return 'bye'
}

asyncPrint('hello world', 2000).then(ret=>{
    console.log(ret);
})
```
await状态转为失败时的处理
```js
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一种写法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  });
}
```

### 同时执行
`await`语句会保持顺序执行，如果不存在顺序关系的两个操作，可以并行执行
```js
//方式一
var [foo, bar] = await Promise.all([getFoo(), getBar()])
//方式二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

### await与for循环
```js
function f(i) {
  return new Promise((res, rej)=>{
    setTimeout(res, 1000*i, i)
  })
}

async function test(){
    for(var i = 0; i<5; i++){
        console.time(i);
        console.log('ret: ' + await f(i));
        console.timeEnd(i)
    }   
}

test()
```
这段输出如下，for循环有被中断的效果
```js
ret: 0
0: 5.512ms
ret: 1
1: 1000.404ms
ret: 2
2: 2000.869ms
ret: 3
3: 3001.408ms
ret: 4
4: 4000.921ms
```

### 错误处理
async/await 从promise迁移过来使用还是很简单的，之前已经总结了部分错误处理

那就是`try...catch`,但每个await写个`try...catch`实在是不够优雅

那就简单写个封装
```js
function to(promise){
    return promise.then(ret=>{
        return [null, ret]
    }).catch(err=>[err])
}

async function test(){
    [err, ret] = await to(getFoo())
    if(err){
        //do something
    }
    [err, ret] = await to(getBar())
    if(err){
        //do something
    }
}
```
虽然也没有多高级，但显然比`try...catch`要优雅的多……

## 总结
本来想从ES6的generator开始总结起，然而ES6规范还没普及就被ES7的async/await干掉了……

不存在兼容性问题的情况下，直接使用async/await能让语义更清晰，也能减少代码量，值得学习和使用

参考资料：
- [async函数](http://es6.ruanyifeng.com/#docs/async)
- [How to write async await without try-catch blocks in Javascript](https://blog.grossman.io/how-to-write-async-await-without-try-catch-blocks-in-javascript/)