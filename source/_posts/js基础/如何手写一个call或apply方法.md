---
title: 如何手写一个call或apply方法
date: 2022-10-23 21:00:00
categories:
  - js基础
tags:
  - js
  - call
  - apply
  - this
---

## call，apply 使用方法与区别

作用：改变 this 指向；
区别：入参形式

```js
const fn = function (arg1, arg2) {
    // do something
};
​
fn.call(this, arg1, arg2); // 参数散列
fn.apply(this, [arg1, arg2]) // 参数使用数组包裹
```

# 实现

- call

```js
Function.prototype.mycall = function (obj) {
  //this 为实际函数，添加this到对象成员，然后删除,可以用symbol实现
  // 处理参数
  obj = obj || window;
  obj.fn = this;

  // es6语法，否则需要Array.prototype.slice.call来处理参数，并且使用eval执行函数
  const args = [...arguments].slice(1);
  const result = obj.fn(...args);
  delete obj.fn;
  return result;
};
```

- apply

```js
Function.prototype.apply_ = function (obj, arr) {
  obj = obj || window;
  obj.fn = this;
  //es6语法
  let result;
  if (arr) {
    result = obj.fn(...arr);
  } else {
    result = obj.fn();
  }
  delete obj.fn;
  return result;
};
```
