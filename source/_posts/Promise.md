---
title: Promise
date: 2020-08-05 21:28:49
tags: [js, Promise]
---


###  Promise三种状态

1. `pending`: 未解决，当调用任意一个任务函数时，暂时处于pending状态，是不会去执行下一个任务的。

2. `resolve`: 当前任务执行完，且执行成功，任务函数内部会手动调用resolve()函数。Promise对象会将任务的状态改为resolve状态。死循环发现状态变为resolve，则自动i++，找到集合中排在当前位移的下一个位置的任务函数执行。并在此将状态改为pending。直到当前任务执行完，才判断是否可以继续。

3. `reject`: 如果当前任务执行过程中出错了，无法继续向后执行，则任务函数中会先手动调用reject()函数。Promise对象将任务的状态转为reject。死循环发现任务的状态变为reject，则不再继续循环，而是退出循环，执行最后一个catch的内容。

###  .then的作用

then不是调用函数的意思，是收集后面任务的作用。

一边收集后面的函数，一边调用第一个函数。

利用状态机制来顺序执行任务
<img src="/images/promise/promise.jpg" style="width:80%;margin: 0 auto;" />
执行过程是由状态来决定的：

> 1. 当调用第一个函数时，状态为pending，（我在执行着呢，后面两个函数给我等着）
> 2. 第一个函数执行完，调用door（resolve）函数（相当于只改变了状态），状态从pending变为fulfilled（办妥   了，解决了）或者resolve （解决）
> 3. 执行第二个ran函数，状态迅速从fulfilled改为pending,当ran里面调door（resolve）时，状态由pending改为fulfilled，接着调用第三个函数
> 4. 在执行函数中，有第二个门 reject，通常用于报告错误用的，reject里面包含错误信息（实参），并传递给catch里面的函数的形参e接收，后续的函数不执行



###  常用的Promise

####  Promise.all()

多个异步任务同时执行，不必按顺序执行

```javascript
Promise.all([work1(), work2(), work3()]).then(....)
```

####  Promise.race()

该方法同样是将多个Promise实例包装成一个新的Promise实例

```javascript
const p = Promise.race([p1, p2, p3]);
```

上面代码中，只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。

栗子：

```javascript
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);

p
.then(console.log)
.catch(console.error);
```

上面代码中，如果 5 秒之内fetch方法无法返回结果，变量p的状态就会变为rejected，从而触发catch方法指定的回调函数。

