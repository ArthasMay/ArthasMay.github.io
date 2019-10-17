---
title: 02 | 作用域链和闭包 ：代码中出现相同的变量，JavaScript引擎是如何选择的？
p: javascript/js-theory-02
date: 2019-10-18 00:06:41
tags:
- JS语言特性
- 原理分析
categories: JavaScript
---

## setTimeout里面的回调函数也是闭包
* 返回值类型Timeout._onTimeout:() => { … }就是回调函数，是一个闭包，所以能捕获作用域中的变量