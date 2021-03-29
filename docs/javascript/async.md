## 异步操作

### 单线程模型

> 单线程模型指的是，JavaScript 只在一个线程上运行。也就是说，JavaScript 同时只能执行一个任务，其他任务都必须在后面排队等待。

> Event Loop, javascript中的一种运行机制，用来解决浏览器单线程的问题。引擎在不停地检查，一遍又一遍，只要同步任务执行完了，引擎就会去检查那些挂起来的异步任务，是不是可以进入主线程了。这种循环检查的机制，就叫做事件循环（Event Loop）

因为 IO 操作（输入输出）很慢（比如 Ajax 操作从网络读取数据），不得不等着结果出来，再往下执行。JavaScript 语言的设计者意识到，这时 CPU 完全可以不管 IO 操作，挂起处于等待中的任务，先运行排在后面的任务。等到 IO 操作返回了结果，再回过头，把挂起的任务继续执行下去。这种机制就是 JavaScript 内部采用的“事件循环”机制（Event Loop）。

### 同步任务和异步任务

程序里面所有的任务，可以分成两类：同步任务（synchronous）和异步任务（asynchronous）。

> 同步任务是那些没有被引擎挂起、在主线程上排队执行的任务。只有前一个任务执行完毕，才能执行后一个任务。

> 异步任务是那些被引擎放在一边，不进入主线程、而进入任务队列的任务。只有引擎认为某个异步任务可以执行了（比如 Ajax 操作从服务器得到了结果），该任务（采用回调函数的形式）才会进入主线程执行。排在异步任务后面的代码，不用等待异步任务结束会马上运行，也就是说，异步任务不具有“堵塞”效应。

### 任务队列和事件循环

JavaScript 运行时，除了一个正在运行的主线程，引擎还提供一个任务队列（task queue），里面是各种需要当前程序处理的异步任务。（实际上，根据异步任务的类型，存在多个任务队列。为了方便理解，这里假设只存在一个队列。）

首先，主线程会去执行所有的同步任务。等到同步任务全部执行完，就会去看任务队列里面的异步任务。如果满足条件，那么异步任务就重新进入主线程开始执行，这时它就变成同步任务了。等到执行完，下一个异步任务再进入主线程开始执行。一旦任务队列清空，程序就结束执行。


### 定时器

> JavaScript 提供定时执行代码的功能，叫做定时器（timer），主要由setTimeout()和setInterval()这两个函数来完成。它们向任务队列添加定时任务。

#### setTimeout()

> setTimeout函数用来指定某个函数或某段代码，在多少毫秒之后执行。

```js
var timerId = setTimeout(func|code, delay);
```

**还有一个需要注意的地方，如果回调函数是对象的方法，那么setTimeout使得方法内部的this关键字指向全局环境，而不是定义时所在的那个对象。**

```js
var x = 1;

var obj = {
  x: 2,
  y: function () {
    console.log(this.x);
  }
};
// this 指向全局了，所以打印结果是外部的x
setTimeout(obj.y, 1000) // 1
```

一种解决方法是将obj.y放入一个函数。

```js
var x = 1;

var obj = {
  x: 2,
  y: function () {
    console.log(this.x);
  }
};

setTimeout(function () {
  obj.y();
}, 1000); // 2
```

另一种解决方法是，使用bind方法，将obj.y这个方法绑定在obj上面。

```js
var x = 1;

var obj = {
  x: 2,
  y: function () {
    console.log(this.x);
  }
};

setTimeout(obj.y.bind(obj), 1000) // 2
```

#### setTimeout()

> setInterval函数的用法与setTimeout完全一致，区别仅仅在于setInterval指定某个任务每隔一段时间就执行一次，也就是无限次的定时执行。

#### clearTimeout()，clearInterval()

> setTimeout和setInterval函数，都返回一个整数值，表示计数器编号。将该整数传入clearTimeout和clearInterval函数，就可以取消对应的定时器。


### Promise 对象

Promise 对象是 JavaScript 的异步操作解决方案，为异步操作提供统一接口。

```js
function f1(resolve, reject) {
  // 异步代码...
}
var p1 = new Promise(f1);
```

Promise 实例有一个then方法，用来指定下一步的回调函数。

```js
var p1 = new Promise(f1);
p1.then(f2);
```

#### Promise 对象的状态

Promise 实例具有三种状态。

* 异步操作未完成（pending）
* 异步操作成功（fulfilled）
* 异步操作失败（rejected）

三种的状态的变化途径只有两种。

* 从“未完成” （pending） 到“成功” （fulfilled）
* 从“未完成” （pending） 到“失败” （rejected）

#### Promise 构造函数 

JavaScript 提供原生的Promise构造函数，用来生成 Promise 实例。

```js
var promise = new Promise(function (resolve, reject) {
  // ...

  if (/* 异步操作成功 */){
    resolve(value);
  } else { /* 异步操作失败 */
    reject(new Error());
  }
});
```

#### Promise.prototype.then() 

Promise 实例的then方法，用来添加回调函数。

then方法可以接受两个回调函数，第一个是异步操作成功时（变为fulfilled状态）的回调函数，第二个是异步操作失败（变为rejected）时的回调函数（该参数可以省略）。一旦状态改变，就调用相应的回调函数。

```js
var p1 = new Promise(function (resolve, reject) {
  resolve('成功');
});
p1.then(console.log, console.error);
// "成功"

var p2 = new Promise(function (resolve, reject) {
  reject(new Error('失败'));
});
p2.then(console.log, console.error);
// Error: 失败
```

#### 微任务

Promise 的回调函数属于异步任务，会在同步任务之后执行。

```js
new Promise(function (resolve, reject) {
  resolve(1);
}).then(console.log);

console.log(2);
// 2
// 1
```

上面代码会先输出2，再输出1。因为console.log(2)是同步任务，而then的回调函数属于异步任务，一定晚于同步任务执行。

但是，Promise 的回调函数不是正常的异步任务，而是微任务（microtask）。它们的区别在于，正常任务追加到下一轮事件循环，微任务追加到本轮事件循环。这意味着，微任务的执行时间一定早于正常任务。

```js
setTimeout(function() {
  console.log(1);
}, 0);

new Promise(function (resolve, reject) {
  resolve(2);
}).then(console.log);

console.log(3);
// 3
// 2
// 1
```

上面代码的输出结果是321。这说明then的回调函数的执行时间，早于setTimeout(fn, 0)。因为then是本轮事件循环执行，setTimeout(fn, 0)在下一轮事件循环开始时执行。