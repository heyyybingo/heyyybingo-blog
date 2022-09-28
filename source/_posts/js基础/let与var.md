---
title: promise原理与实现
date: 2022-09-27 21:00:00
categories:
  - js基础
tags:
  - js
  - let
  - var
---

# 区别

var 声明是全局作用域或函数作用域，而 let 和 const 是块作用域。 var 变量可以在其范围内更新和重新声明； let 变量可以被更新但不能重新声明； const 变量既不能更新也不能重新声明。 它们都被提升到其作用域的顶端

# let 如何转成 var

https://juejin.cn/post/6907207880022654984
