---
title: 如何手写一个bind方法
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

作用：

- 可以修改函数 this 指向。
- bind 返回一个绑定了 this 的新函数 bindedFcuntion
- 支持函数柯里化，我们在返回 bound 函数时已传递了部分参数 2，在调用时 bindedFcuntion 补全了剩余参数。
- bindedFcuntion 的 this 无法再被修改，使用 call、apply 也不行。

# 实现

不考虑 new 的情况

```js
Function.prototype.mybind = function (obj) {
  // 变量保存函数
  const fn = this;
  //获取当前参数
  const args = Array.prototype.slice.call(arguments, 1);
  return function () {
    // 函数柯里化，拼接参数
    const params = args.concat(Array.prototype.slice.call(arguments));
    fn.apply(obj);
  };
};
```

考虑 new 的情况，需要考虑 new 的时候，this 的指向，以及原型的继承

```js
Function.prototype.mybind = function (obj) {
  // 变量保存函数
  const fn = this;
  //获取当前参数
  const args = Array.prototype.slice.call(arguments, 1);
  const binded = function () {
    // 函数柯里化，拼接参数
    const params = args.concat(Array.prototype.slice.call(arguments));
    fn.apply(this.contructor === fn ? this : obj, params);
  };
  binded.prototype = fn.prototype;
};
```
