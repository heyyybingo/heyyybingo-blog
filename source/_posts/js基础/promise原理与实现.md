---
title: promise原理与实现
date: 2022-09-05 21:00:00
categories:
  - js
tags:
  - js
  - 异步编程
  - promise
---

# 异步编程背景

js 引擎建立在单线程事件循环的概念上，JS 引擎在同一时刻只能执行一段代码

- 事件模型:
  当用户点击一个按钮或按下键盘上的一个键时，一个事件（ event ）——例如 onclick —— 就被触发了;当 button 被点 击，赋值给 onclick 的函数就被添加到作业队列的尾部，并在队列前部所有任务结束之后再 执行。

```js
let button = document.getElementById("my-btn");
button.onclick = function (event) {
  console.log("Clicked");
};
```

- 回调模式:
  当 Node.js 被创建时，它通过普及回调函数编程模式提升了异步编程模型。回调函数模式类 似于事件模型，因为异步代码也会在后面的一个时间点才执行。不同之处在于需要调用的函 数（即回调函数）是作为参数传入的，

```js
readFile("example.txt", function (err, contents) {
  if (err) {
    throw err;
  }
  console.log(contents);
});
console.log("Hi!");
```

- Promise 是为异步操作的结果所准备的占位符。函数可以返回一个 Promise，而不必订阅一个 事件或向函数传递一个回调参数，就

# 生命周期

## pending

一个挂起的 Promise 也被认为是未决的（ unsettled ）。上个例子中的 Promise 在 readFile() 函数返回它的时候就是处在挂起态。一旦异步操作结束， Promise 就会被认为是已决的（ settled ），并进入两种可能状态之一

## fulfilled

Promise 的异步操作已成功结束；

## rejected

Promise 的异步操作未成功结束，可能是一个错误，或由其他原 因导致

# 串联的 promise

# 运行多个 promise

## Promise.all

Promise.all() 方法接收单个可迭代对象（如数组）作为参数，并返回一个 Promise 。这个可迭代对象的元素都是 Promise ，只有在它们都完成后，所返回的 Promise 才会被完成。

## Promise.race

Promise.race() 提供了监视多个 Promise 的一个稍微不同的方法。此方法也接受一个包含需监视的 Promise 的可迭代对象，并返回一个新的 Promise ，但一旦来源 Promise 中有一个被解决，所返回的 Promise 就会立刻被解决。与等待所有 Promise 完成的 Promise.all() 方法 不同，在来源 Promise 中任意一个被完成时， Promise.race() 方法所返回的 Promise 就能 作出响应

# 简易实现

```js
enum PromiseState{
  FULLFILLED="fullfilled",
  PENDING="pending",
  REJECTED="rejected"
}
class MyPromise {
  //promise的状态
  promiseState = PromiseState.PENDING;
  //promise的结果
  promiseResult = null;
  // 成功的回调函数数组
  onFullfilledCallbacks = [];
  // 失败的回调函数数组
  onRejectedCallbacks = [];
  static resolve(val) {
    return new MyPromise((resolve) => resolve(val));
  }
  static reject(val) {
    return new MyPromise((_, reject) => reject(val));
  }
  static all(pArr) {
    const result = [];
    let successCount = 0;
    return new MyPromise((resolve, reject) => {
        //TODO 判断promise类型
      pArr.forEach((p, index) => {
        p.then(
          (v) => {
            successCount++;

            result[index] = v;
            if (successCount === pArr.length) {
              resolve(result);
            }
          },
          (e) => {
            reject(e);
          }
        );
      });
    });
  }
  static race(pArr) {
    return new MyPromise((resolve, reject) => {
        //TODO 判断promise类型
      pArr.forEach((p, index) => {
        p.then(
          (v) => {
            resolve(v);
          },
          (e) => {
            reject(e);
          }
        );
      });
    });
  }
  constructor(executor) {
    this.initBind();
    try {
      //拦截错误
      executor(this.selfResolve, this.selfReject);
    } catch (e) {
      this.reject(e);
    }
  }
  //绑定函数指向
  initBind() {
    this.selfResolve = this.selfResolve.bind(this);
    this.selfReject = this.selfReject.bind(this);
  }
  selfResolve(val) {
    if (this.promiseState !== PromiseState.PENDING) {
      return;
    }
    this.promiseState = PromiseState.FULLFILLED;
    this.promiseResult = val;
    //执行then注册任务的回调
    while (this.onFullfilledCallbacks.length) {
      this.onFullfilledCallbacks.shift()(this.promiseResult);
    }
  }
  selfReject(reason) {
    if (this.promiseState !== PromiseState.PENDING) {
      return;
    }
    this.promiseState = PromiseState.REJECTED;
    this.promiseResult = reason;
    while (this.onRejectedCallbacks.length) {
      this.onRejectedCallbacks.shift()(this.promiseResult);
    }
  }
  then(onFullfilled, onRejected) {
    onFullfilled =
      typeof onFullfilled === "function" ? onFullfilled : (val) => val;
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (reason) => {
            throw reason;
          };

    // 不考虑链式传递，对promise结果处理
    // if (this.promiseState === PromiseState.FULLFILLED) {
    //   onFullfilled(this.promiseResult);
    // } else if (this.promiseState === PromiseState.REJECTED) {
    //   onRejected(this.promiseResult);
    // } else if (this.promiseState === PromiseState.PENDING) {
    //   this.onFullfilledCallbacks.push(onFullfilled);
    //   this.onRejectedCallbacks.push(onRejected);
    // }

    //实现链式传递，返回一个promise
    return new MyPromise((resolve, reject) => {
      const resolvePromise = (cb) => {
       //未任务，保证不是立即执行的
       setTimeout(()=>{
         try {
           //取回调执行结果
          const cbResult = cb(this.promiseResult);

          if (cbResult instanceof MyPromise) {
            cbResult.then(resolve, reject);
          } else {
            resolve(cbResult);
          }
        } catch (e) {
          //抛出异常
          reject(e);
        }
       })
      };
      if (this.promiseState === PromiseState.FULLFILLED) {
        onFullfilled(this.promiseResult);
      } else if (this.promiseState === PromiseState.REJECTED) {
        onRejected(this.promiseResult);
      } else if (this.promiseState === PromiseState.PENDING) {
        this.onFullfilledCallbacks.push(onFullfilled);
        this.onRejectedCallbacks.push(onRejected);
      }
    });
  }
}

```

## Promise.all

## Promise.race
